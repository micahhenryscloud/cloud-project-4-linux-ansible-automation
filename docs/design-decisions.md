# Design Decisions

This document explains the key design choices made in the Linux and Ansible automation project and the reasoning behind them.

## 1. Linux as the Base Operating System

Linux was used because most cloud servers run on Linux-based operating systems.

Working directly with Linux helped develop practical skills in:

- Filesystem navigation
- Service management
- Process inspection
- Log inspection
- Network troubleshooting
- Permissions
- Users and groups

These are core skills for cloud engineering because cloud infrastructure often requires troubleshooting at the operating system level.

## 2. Apache as the Test Web Server

Apache was used as the example service because it is a common Linux web server and has a clear service lifecycle.

It allowed me to practise:

```text
Install service
   ↓
Start service
   ↓
Stop service
   ↓
Check service status
   ↓
Verify web response
```

This made it useful for learning how Linux services behave and how to recover them when they fail.

## 3. Systemd for Service Management

Systemd was used to manage the Apache service.

The key commands used were:

```bash
systemctl status apache2
systemctl is-active apache2
systemctl stop apache2
systemctl start apache2
```

This was important because cloud engineers often need to check whether an application service is running correctly on a server.

## 4. Logs for Troubleshooting

Apache access logs and error logs were used to understand application behaviour.

The key log files were:

```text
/var/log/apache2/access.log
/var/log/apache2/error.log
```

Access logs show incoming requests.

Error logs show service or application problems.

This helped demonstrate the difference between:

```text
The service is running
```

and:

```text
The application is behaving correctly
```

## 5. Port and Process Inspection

Linux process and port inspection were used to connect services to the network.

Commands such as:

```bash
ps -ef | grep apache2
ss -tulpn | grep :80
```

were used to verify that Apache was running and listening on port 80.

This was important because a service can only receive web traffic if it is actively listening on the correct port.

The troubleshooting chain was:

```text
Service
   ↓
Process
   ↓
Port
   ↓
HTTP Response
```

## 6. Permissions and Ownership

Linux file permissions were explored using:

```bash
ls -l
chmod
whoami
id
groups
```

This was important because many Linux issues are caused by incorrect ownership or permissions.

For example, trying to create a file in a root-owned directory resulted in:

```text
Permission denied
```

This helped demonstrate that access depends on:

```text
User
+
Group
+
Permissions
=
Access
```

## 7. Users, Groups and sudo

A test Linux user was created to practise user management.

Commands included:

```bash
useradd
passwd
id
groups
sudo
```

This helped demonstrate how Linux separates normal users from privileged users.

The key model was:

```text
Normal User
   ↓
sudo
   ↓
Root Privileges
```

This is important because cloud engineers need to understand how privilege escalation works without giving every user unrestricted access by default.

## 8. Ansible for Configuration Management

Ansible was used to automate Linux server configuration.

Instead of manually running commands one by one, Ansible allowed the desired server state to be described in a playbook.

The manual process would be:

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
```

The Ansible process became:

```bash
sudo ansible-playbook install-apache.yml
```

This demonstrates how configuration management tools reduce repetitive manual work.

## 9. Playbook-Based Automation

The Ansible playbook was used to:

- Update the apt package cache
- Install Apache
- Start Apache
- Enable Apache on boot
- Deploy a custom homepage

This created a repeatable automation workflow.

The model was:

```text
Ansible Playbook
   ↓
Linux Server
   ↓
Apache Installed
   ↓
Service Running
   ↓
Homepage Deployed
```

## 10. Idempotency

Ansible was used because it is idempotent.

This means the playbook can be run multiple times without repeatedly making unnecessary changes.

If Apache is already installed, Ansible does not reinstall it unnecessarily.

If the service is already running, Ansible leaves it running.

This is important because infrastructure automation should be safe to re-run.

The model is:

```text
Desired State Defined
   ↓
Ansible Checks Current State
   ↓
Only Applies Needed Changes
```

## 11. Repairing Configuration Drift

The Apache service was deliberately stopped and then the Ansible playbook was re-run.

Ansible restored the service to the desired state by starting Apache again.

This demonstrated configuration drift repair:

```text
Desired State: Apache running
Actual State: Apache stopped
   ↓
Run Ansible
   ↓
Apache restored
```

This is one of the main reasons configuration management tools are used in real environments.

## Summary

The project was designed around four main principles:

- Understand Linux troubleshooting fundamentals
- Learn how services, processes, ports and logs connect
- Understand users, permissions and sudo
- Use Ansible to automate repeatable server configuration

This project demonstrates the connection between Linux administration and infrastructure automation.