# Troubleshooting Notes

This document records real Linux and Ansible issues encountered during the project, how they were diagnosed, and how they were fixed.

## Issue 1: Permission denied when creating a folder

### Problem

Creating a folder failed with:

```text
Permission denied
```

### Investigation

I checked the current directory and its permissions using:

```bash
pwd
ls -ld .
```

The directory was owned by `root`, and the current user did not have write permission.

### Root Cause

The command was being run in a directory where the normal user did not have permission to create files or folders.

### Fix

I moved into the user home directory:

```bash
cd ~
```

Then created the folder successfully.

### Lesson Learned

Creating files or folders requires write permission on the current directory. If a directory is owned by root and the current user does not have write permission, Linux will return `Permission denied`.

---

## Issue 2: nginx.service could not be found

### Problem

Stopping Nginx failed with:

```text
Unit nginx.service could not be found
```

### Investigation

I checked whether Nginx was installed using:

```bash
which nginx
```

No path was returned.

### Root Cause

Nginx was not installed on the server.

### Fix

I installed Apache instead and used it as the test web server:

```bash
sudo apt update -y
sudo apt install apache2 -y
```

### Lesson Learned

Before troubleshooting a service, confirm that the software is actually installed and registered as a systemd service.

---

## Issue 3: systemctl status appeared to trap the terminal

### Problem

After running:

```bash
systemctl status apache2
```

the terminal appeared not to accept new commands.

### Investigation

The command output had opened in a pager.

### Root Cause

`systemctl status` can display output through a pager such as `less`.

### Fix

I pressed:

```text
q
```

to exit the pager.

### Lesson Learned

Some Linux commands open interactive viewers. If the terminal appears stuck inside command output, try exiting with `q`.

---

## Issue 4: Apache stopped and port 80 disappeared

### Problem

After stopping Apache, no process appeared to be listening on port 80.

### Investigation

I stopped Apache:

```bash
sudo systemctl stop apache2
```

Then checked port 80:

```bash
sudo ss -tulpn | grep :80
```

No output was returned.

### Root Cause

Apache was no longer running, so no process was listening on port 80.

### Fix

I started Apache again:

```bash
sudo systemctl start apache2
```

Then verified port 80 returned:

```bash
sudo ss -tulpn | grep :80
```

### Lesson Learned

A web server must be running and listening on the correct port before it can serve HTTP traffic.

---

## Issue 5: Apache process was killed but restarted automatically

### Problem

The Apache master process was killed manually using `kill -9`.

### Investigation

I checked Apache activity using:

```bash
systemctl is-active apache2
ps -ef | grep apache2
```

Apache still showed as active, but the process IDs had changed.

### Root Cause

Apache was managed by systemd. When the process was killed, systemd restarted it and created new Apache processes.

### Fix

No manual fix was required because systemd recovered the service.

### Lesson Learned

Systemd manages services and can restart processes. This is similar in principle to Auto Scaling replacing EC2 instances or Kubernetes replacing Pods.

---

## Issue 6: Access logs showed ELB health checks

### Problem

Apache access logs showed repeated requests from `ELB-HealthChecker/2.0`.

### Investigation

I checked the Apache access log:

```bash
sudo tail -5 /var/log/apache2/access.log
```

The log showed successful HTTP 200 responses to ELB health checks.

### Root Cause

The AWS Application Load Balancer was checking whether the EC2 instance was healthy.

### Fix

No fix was needed. The log entries confirmed the ALB health checks were reaching Apache successfully.

### Lesson Learned

Application logs can reveal infrastructure behaviour. In this case, the Apache logs showed AWS load balancer health checks in real time.

---

## Issue 7: Permission denied when creating a test file

### Problem

Creating a file with:

```bash
touch myfile.txt
```

failed with:

```text
Permission denied
```

### Investigation

I checked the current directory permissions:

```bash
ls -ld .
```

The directory was owned by `root`.

### Root Cause

The current user did not have write permission in that directory.

### Fix

I moved to the home directory:

```bash
cd ~
```

Then created the file successfully.

### Lesson Learned

File creation depends on directory permissions. Even if a file does not exist yet, Linux checks whether the user can write to the directory.

---

## Issue 8: File permissions removed access to a file

### Problem

After changing file permissions with:

```bash
chmod 000 myfile.txt
```

the file showed:

```text
----------
```

### Investigation

I checked the file permissions:

```bash
ls -l myfile.txt
```

The file had no read, write or execute permissions for owner, group or others.

### Root Cause

`chmod 000` removed all permissions from the file.

### Fix

I restored normal file permissions:

```bash
chmod 644 myfile.txt
```

This returned the file to:

```text
-rw-r--r--
```

### Lesson Learned

Linux file access is controlled by permission bits. A user can own a file but still be unable to read it if permissions are removed.

---

## Issue 9: Ansible localhost unreachable due temporary directory issue

### Problem

Ansible returned an unreachable error when trying to run against localhost.

The error referenced:

```text
/root/.ansible/tmp
```

### Investigation

Ansible was trying to use a temporary directory under `/root`.

### Root Cause

The Ansible temporary directory was not accessible in the current execution context.

### Fix

I set Ansible to use a temporary directory under `/tmp`:

```bash
export ANSIBLE_LOCAL_TEMP=/tmp/.ansible/tmp
mkdir -p /tmp/.ansible/tmp
```

Then ran Ansible with:

```bash
ansible localhost -m ping -c local -i localhost, -e "ansible_remote_tmp=/tmp/.ansible/tmp"
```

I also created an `ansible.cfg` file:

```ini
[defaults]
remote_tmp = /tmp/.ansible/tmp
local_tmp = /tmp/.ansible/tmp
```

### Lesson Learned

Ansible uses temporary directories when executing tasks. If those directories are not writable, Ansible can fail before running the actual module.

---

## Issue 10: Ansible apt task failed due permission denied

### Problem

The Ansible playbook failed on the apt update task with a permission error.

The error referenced the apt lock file:

```text
Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)
```

### Investigation

The playbook was attempting to run package management tasks without sufficient privileges.

### Root Cause

Installing packages and updating apt requires root privileges.

### Fix

I ran the playbook with sudo:

```bash
sudo ansible-playbook install-apache.yml
```

The playbook then completed successfully.

### Lesson Learned

Package installation and service configuration usually require elevated privileges. Ansible tasks that modify system state often need `become: yes` and may need the playbook to be run with sudo depending on the environment.

---

## Issue 11: Ansible restored Apache after it was stopped

### Problem

Apache was manually stopped to test whether Ansible could repair configuration drift.

### Investigation

I stopped Apache:

```bash
sudo systemctl stop apache2
```

Then confirmed it was inactive:

```bash
systemctl is-active apache2
```

### Root Cause

The actual server state no longer matched the desired state in the Ansible playbook.

### Fix

I reran:

```bash
sudo ansible-playbook install-apache.yml
```

Ansible started Apache again and restored the desired state.

### Lesson Learned

Ansible is useful for repairing configuration drift. If the server moves away from the desired state, rerunning the playbook can bring it back into compliance.

---

## Summary

The main troubleshooting themes in this project were:

- Linux directory and file permissions
- Service management with systemd
- Process and port troubleshooting
- Reading application logs
- Understanding users, groups and sudo
- Fixing Ansible execution environment issues
- Running Ansible with the correct privileges
- Using Ansible to repair configuration drift

These issues helped connect Linux administration with practical infrastructure automation.