# 100 Days of DevOps

I am learning DevOps from the basics through hands-on daily tasks, inspired by the KodeKloud 100 Days of DevOps challenge.

My goal is not just to memorize commands. For every task, I try to understand the concept, perform the change on a real Linux server, troubleshoot issues, verify the result, and document what I learned.

---

## My Learning Approach

```text
Understand the concept
        ↓
Read the requirement
        ↓
Do the task
        ↓
Troubleshoot if needed
        ↓
Verify the result
        ↓
Document what I learned
        ↓
Push to GitHub
```

---

## Lab Environment

I am mainly using AWS EC2 instances for hands-on practice.

### Operating Systems
- Ubuntu 26.04 LTS
- Red Hat Enterprise Linux 10

### Tools used so far
- Linux
- AWS EC2
- SSH
- Git
- GitHub
- Cron
- SELinux
- Ansible
- MariaDB

More tools will be added as the challenge progresses.

---

# Progress

| Day | Topic | What I Practiced | Status |
| --- | --- | --- | --- |
| [Day 1](./day-01-linux-service-user/) | Linux Service User | Created a service user with a non-interactive shell using `nologin` | ✅ |
| [Day 2](./day-02-temporary-user-expiry/) | Temporary User Expiry | Created a temporary Linux user and configured account expiry | ✅ |
| [Day 3](./day-03-disable-root-ssh-login/) | SSH Security | Disabled direct root SSH login and verified normal sudo access | ✅ |
| [Day 4](./day-04-script-execution-permissions/) | Linux Permissions | Learned `r`, `w`, `x`, ownership, groups, `chmod`, and script execution | ✅ |
| [Day 5](./day-05-selinux-configuration/) | SELinux | Learned Enforcing, Permissive, persistent configuration, and boot settings | ✅ |
| [Day 6](./day-06-cron-job/) | Cron Jobs | Created and verified a cron job that runs every 5 minutes | ✅ |
| [Day 7](./day-07-ssh-key-authentication/) | SSH Key Authentication | Configured passwordless SSH between Ubuntu and RHEL EC2 instances | ✅ |
| [Day 8](./day-08-ansible-inventory-and-ad-hoc-commands/) | Ansible | Created an inventory, fixed SSH authentication issues, used Ansible ping, and ran remote commands | ✅ |
| [Day 9](./day-09-mariadb-troubleshooting/) | MariaDB Troubleshooting | Created a safe MariaDB failure, diagnosed the issue, fixed it, and verified database health | ✅ |

**Current Progress: 9 / 100 Days**

---

# What I Have Learned So Far

## Day 1 - Linux Service User

I learned that Linux users are not only for humans.

Applications and services can also have their own users.

Instead of running services as `root`, it is safer to create a dedicated service account with only the permissions it needs.

I created:

```text
backupsvc
```

with a non-interactive shell:

```text
/usr/sbin/nologin
```

The main idea was:

```text
Service needs an identity
        ↓
Does not need a terminal
        ↓
Use a service account
        ↓
Use nologin
```

I also learned:

```bash
command -v nologin
```

to find where `nologin` exists, and:

```bash
getent passwd backupsvc
```

to verify the user configuration.

---

## Day 2 - Temporary User Expiry

I learned how to create a temporary Linux account and configure an expiry date.

This is useful for:

- contractors
- temporary developers
- consultants
- short-term access

Instead of relying on someone to remember to delete the account later, Linux can block access automatically after the expiry date.

I used:

```bash
useradd tempdev
```

and:

```bash
chage -E 2026-09-15 tempdev
```

Then verified using:

```bash
chage -l tempdev
```

The main lesson was:

```text
Temporary access
      ↓
Set expiry date
      ↓
Linux handles it automatically
```

---

## Day 3 - Disable Direct Root SSH Login

I learned why allowing direct SSH login as `root` is risky.

`root` has full control of the server, so a safer approach is:

```text
SSH as normal user
        ↓
Use sudo when needed
```

I configured:

```text
PermitRootLogin no
```

and verified the effective SSH configuration with:

```bash
sshd -T | grep permitrootlogin
```

I also tested both:

```text
ubuntu SSH login → worked
root SSH login   → blocked
```

The biggest lesson was to verify SSH changes before closing the current session.

---

## Day 4 - Linux File and Script Permissions

I learned how Linux file permissions work.

Permissions are split into:

```text
owner
group
others
```

and use:

```text
r = read
w = write
x = execute
```

For example:

```text
-rwxr-xr-x
```

means:

```text
Owner  → read + write + execute
Group  → read + execute
Others → read + execute
```

I created a shell script and saw that without execute permission:

```bash
./day4.sh
```

returned:

```text
Permission denied
```

Then I added execute permission with:

```bash
chmod u+x day4.sh
```

and the script worked.

I also learned about the shebang:

```bash
#!/bin/bash
```

which tells Linux to run the script using Bash.

---

## Day 5 - SELinux

I learned that SELinux adds another security layer on top of normal Linux permissions.

The flow is:

```text
Application
    ↓
Linux permissions
    ↓
SELinux policy
    ↓
Allow or Deny
```

I learned the difference between:

```text
Enforcing  → block + log
Permissive → allow + log
Disabled   → SELinux not enforcing
```

Useful commands:

```bash
sestatus
getenforce
sudo setenforce 0
sudo setenforce 1
```

I also learned that:

```text
setenforce
```

changes the current mode temporarily.

The persistent configuration is related to:

```text
/etc/selinux/config
```

On RHEL 10, I also learned how the boot argument:

```text
selinux=0
```

can be configured using:

```bash
sudo grubby --update-kernel ALL --args selinux=0
```

The important lesson was understanding the difference between:

```text
current running state
```

and:

```text
state after reboot
```

---

## Day 6 - Cron Jobs

I learned how Linux can automatically run commands on a schedule using cron.

A cron expression has five time fields:

```text
* * * * *
│ │ │ │ │
│ │ │ │ └── day of week
│ │ │ └──── month
│ │ └────── day of month
│ └──────── hour
└────────── minute
```

I learned:

```text
*   → every value
*/5 → every 5 units
```

I created this cron job:

```bash
*/5 * * * * echo hello > /tmp/cron_text
```

This runs every 5 minutes.

I verified:

```bash
systemctl is-active cron
```

and checked the actual result using:

```bash
cat /tmp/cron_text
```

The biggest lesson was:

> Automation is not complete until I verify that it actually ran.

---

## Day 7 - SSH Key Authentication

I learned how passwordless SSH works using public and private keys.

The private key stays on the source machine:

```text
~/.ssh/id_ed25519
```

The public key can be copied to the destination server:

```text
~/.ssh/id_ed25519.pub
```

The destination server stores allowed public keys in:

```text
~/.ssh/authorized_keys
```

The flow is:

```text
Source server
Private key
     ↓
SSH
     ↓
Destination server
authorized_keys
     ↓
Matching public key
     ↓
Login allowed
```

I also learned how to understand SSH errors.

```text
Connection timed out
→ network / security group issue
```

```text
Permission denied (publickey)
→ SSH authentication issue
```

This troubleshooting difference was very useful.

---

## Day 8 - Ansible

I learned the basic Ansible architecture.

```text
Control Node
     ↓
Inventory
     ↓
SSH
     ↓
Managed Node
```

My Ubuntu EC2 acted as the control node.

My RHEL EC2 acted as the managed node.

I created an inventory:

```text
[rhel]
172.31.27.84 ansible_user=ec2-user ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_ed25519
```

I tested Ansible using:

```bash
ansible all -i inventory -m ping
```

and got:

```text
"ping": "pong"
```

Then I ran a real remote command:

```bash
ansible rhel -i inventory -m command -a "hostname"
```

I learned that Ansible depends on several things working together:

```text
Inventory
   ↓
Network
   ↓
SSH
   ↓
SSH key
   ↓
Python
   ↓
Ansible module
```

---

## Day 9 - MariaDB Troubleshooting

I learned how to troubleshoot a Linux service instead of randomly trying fixes.

My troubleshooting process was:

```text
Service fails
    ↓
Check status
    ↓
Read the exact error
    ↓
Find root cause
    ↓
Fix only the problem
    ↓
Start service
    ↓
Verify
```

I installed MariaDB and confirmed it was healthy using:

```bash
systemctl status mariadb
```

and:

```bash
mariadb-admin ping
```

Then I intentionally created a bad configuration:

```text
invalid_option_for_day9=1
```

MariaDB failed with:

```text
unknown variable 'invalid_option_for_day9=1'
```

I found the problem using:

```bash
systemctl status mariadb --no-pager -l
```

Then I removed only the broken config file and started MariaDB again.

Final verification:

```text
Active: active (running)
```

and:

```text
mysqld is alive
```

The biggest lesson was:

```text
Observe
   ↓
Diagnose
   ↓
Fix
   ↓
Verify
```

---

# Repository Structure

```text
100-days-of-devops/
├── README.md
├── day-01-linux-service-user/
│   └── README.md
├── day-02-temporary-user-expiry/
│   └── README.md
├── day-03-disable-root-ssh-login/
│   └── README.md
├── day-04-script-execution-permissions/
│   └── README.md
├── day-05-selinux-configuration/
│   └── README.md
├── day-06-cron-job/
│   └── README.md
├── day-07-ssh-key-authentication/
│   └── README.md
├── day-08-ansible-inventory-and-ad-hoc-commands/
│   └── README.md
└── day-09-mariadb-troubleshooting/
    └── README.md
```

Each day contains:

```text
Task
Concept
Commands
Errors I faced
Troubleshooting
Verification
What I learned
```

---

# My Troubleshooting Mindset

One of the main things I am trying to build during this challenge is a good troubleshooting habit.

Instead of:

```text
Something broke
    ↓
Try random commands
```

I want to work like this:

```text
Something broke
    ↓
Observe
    ↓
Read logs / errors
    ↓
Understand the problem
    ↓
Fix the exact cause
    ↓
Verify
```

This mindset has already helped with:

- SSH issues
- security groups
- Ansible authentication
- MariaDB failures

---

# Main Goal

By the end of these 100 days, I want to be comfortable with real DevOps work, not just definitions.

I want to be able to:

```text
Read a requirement
      ↓
Understand what needs to happen
      ↓
Choose the right tool
      ↓
Make the change
      ↓
Troubleshoot failures
      ↓
Verify the result
      ↓
Explain what I did clearly
```

The goal is to gradually build skills in:

- Linux
- Networking
- Git
- GitHub
- Bash
- AWS
- Ansible
- Docker
- CI/CD
- Terraform
- Kubernetes
- Helm
- Monitoring
- Logging
- Security
- Troubleshooting

---

## Current Progress

```text
Day 1  ✅
Day 2  ✅
Day 3  ✅
Day 4  ✅
Day 5  ✅
Day 6  ✅
Day 7  ✅
Day 8  ✅
Day 9  ✅
```

**9 / 100 Days Completed**

---

> Learning DevOps one real task at a time.