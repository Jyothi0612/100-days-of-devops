# Day 10 - Automate Website Backup with Bash and SCP

## What was the task?

The goal of Day 10 was to automate a website backup using a Bash script.

The backup process had two main parts:

```text
Create ZIP backup
        ↓
Copy ZIP to another server
```

The source directory was:

```text
/var/www/html/media
```

The local backup file was:

```text
/backup/xfusioncorp_media.zip
```

Then the archive was copied to another EC2 instance using `scp`.

---

## Why is a remote backup important?

At first, it may seem enough to create a backup on the same server.

For example:

```text
App Server
├── website
└── backup.zip
```

But if the whole server or disk is lost:

```text
Server lost
   ↓
Website lost
   +
Backup lost
```

So a better design is:

```text
Source Server
     ↓
Create backup
     ↓
Copy backup
     ↓
Backup Server
```

This way, the backup is stored separately from the original data.

---

## Lab Environment

I used two Ubuntu EC2 instances.

### Source server

```text
172.31.29.145
```

This server contained the website files and the backup script.

### Backup server

```text
172.31.18.142
```

This server received the backup archive.

The flow looked like this:

```text
Source Server
172.31.29.145
      ↓
/var/www/html/media
      ↓
Bash script
      ↓
/backup/xfusioncorp_media.zip
      ↓
scp over SSH
      ↓
Backup Server
172.31.18.142
      ↓
/tmp/xfusioncorp_media.zip
```

---

## Checking the source directory

I first checked whether the expected source directory existed:

```bash
ls -ld /var/www/html/media
```

The output was:

```text
ls: cannot access '/var/www/html/media': No such file or directory
```

This meant the clean EC2 server did not already have the KodeKloud-style directory.

So I created it.

---

## Creating the source directory

I ran:

```bash
mkdir -p /var/www/html/media
```

`mkdir` creates a directory.

The `-p` option means:

```text
create parent directories if they do not already exist
```

So Linux could create:

```text
/var
└── www
    └── html
        └── media
```

---

## Creating sample website files

To have something real to back up, I created two sample files:

```bash
touch /var/www/html/media/image1.jpg /var/www/html/media/image2.jpg
```

The source directory then looked like:

```text
/var/www/html/media/
├── image1.jpg
└── image2.jpg
```

---

## Creating the backup directory

I created:

```bash
mkdir -p /backup
```

This directory would store the ZIP archive locally before it was copied to the backup server.

The archive path was:

```text
/backup/xfusioncorp_media.zip
```

---

## Checking whether ZIP was installed

Before writing the script, I checked whether the `zip` command was available:

```bash
zip -v
```

The output confirmed:

```text
This is Zip 3.0
```

So the server had the tool needed to create ZIP archives.

---

# Testing the Backup Manually First

Before automating anything, I tested the backup command manually.

I ran:

```bash
zip -r /backup/xfusioncorp_media.zip /var/www/html/media
```

The output showed:

```text
adding: var/www/html/media/
adding: var/www/html/media/image1.jpg
adding: var/www/html/media/image2.jpg
```

This confirmed the archive was created successfully.

---

## Understanding the ZIP command

The command was:

```bash
zip -r /backup/xfusioncorp_media.zip /var/www/html/media
```

Breakdown:

```text
zip
→ create a ZIP archive
```

```text
-r
→ recursively include the directory and its contents
```

```text
/backup/xfusioncorp_media.zip
→ destination archive
```

```text
/var/www/html/media
→ source directory
```

The main lesson was:

```text
Test command manually
        ↓
Confirm it works
        ↓
Put it into a script
```

---

## Verifying the ZIP file

I checked the archive with:

```bash
ls -lh /backup/xfusioncorp_media.zip
```

The output was:

```text
-rw-r--r-- 1 root root 560 Sep 10 19:55 /backup/xfusioncorp_media.zip
```

This confirmed the file existed.

---

## Verifying the ZIP contents

I also checked what was actually inside the archive:

```bash
unzip -l /backup/xfusioncorp_media.zip
```

The output included:

```text
var/www/html/media/
var/www/html/media/image1.jpg
var/www/html/media/image2.jpg
```

So I knew the backup contained the expected files.

This was important because:

```text
Backup file exists
```

does not always mean:

```text
Backup contains correct data
```

So I verified both.

---

# Creating the Bash Script

I created a directory for scripts:

```bash
mkdir -p /scripts
```

Then I created:

```text
/scripts/media_backup.sh
```

using:

```bash
nano /scripts/media_backup.sh
```

The first version of the script was:

```bash
#!/bin/bash

zip -r /backup/xfusioncorp_media.zip /var/www/html/media
```

---

## What is the shebang?

The first line was:

```bash
#!/bin/bash
```

This is called a shebang.

It tells Linux:

```text
Run this script using Bash
```

Without it, Linux may not know which interpreter should execute the file.

---

## Making the script executable

I ran:

```bash
chmod +x /scripts/media_backup.sh
```

This gave the script execute permission.

Then I ran it directly:

```bash
/scripts/media_backup.sh
```

The output showed:

```text
updating: var/www/html/media/
updating: var/www/html/media/image1.jpg
updating: var/www/html/media/image2.jpg
```

The word:

```text
updating
```

appeared because the ZIP archive already existed from the manual test.

So the script updated the existing archive.

---

# Configuring SSH Key Authentication

The second half of the task was to copy the backup to another server without entering a password.

The destination server was:

```text
172.31.18.142
```

From the source server, I first tested:

```bash
ssh ubuntu@172.31.18.142
```

It failed with:

```text
Permission denied (publickey)
```

This told me:

```text
Network connection worked
SSH service responded
Authentication failed
```

So the problem was not networking.

It was SSH key authentication.

---

## Checking for an existing SSH key

On the source server, I checked:

```bash
ls -l /root/.ssh
```

I found:

```text
id_ed25519
id_ed25519.pub
```

These are:

```text
id_ed25519
→ private key
```

```text
id_ed25519.pub
→ public key
```

So I did not need to generate another key pair.

---

## Viewing the public key

I ran:

```bash
cat /root/.ssh/id_ed25519.pub
```

The public key was:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHe6CEiLdBLu6jlkZXhRb/kmHV7KuxFuh4o+mV9lpLF8 root@ip-172-31-29-145
```

The public key is safe to copy to another server.

The private key should never be shared.

---

## Adding the public key to the backup server

On the backup server, I added the source server's public key to:

```text
/home/ubuntu/.ssh/authorized_keys
```

The idea was:

```text
Source server public key
        ↓
Backup server authorized_keys
        ↓
Backup server trusts source server
```

After this, I tested SSH again:

```bash
ssh ubuntu@172.31.18.142
```

This time the connection worked without asking for a password.

---

# Testing SCP Manually

Before adding `scp` to the Bash script, I tested it manually.

I ran:

```bash
scp /backup/xfusioncorp_media.zip ubuntu@172.31.18.142:/tmp/
```

This copied:

```text
/backup/xfusioncorp_media.zip
```

from the source server to:

```text
/tmp/xfusioncorp_media.zip
```

on the backup server.

---

## Verifying the remote copy

On the backup server, I checked:

```bash
ls -lh /tmp/xfusioncorp_media.zip
```

The output was:

```text
-rw-r--r-- 1 ubuntu ubuntu 560 Sep 10 20:07 /tmp/xfusioncorp_media.zip
```

This confirmed the archive had actually reached the second server.

---

# Final Bash Script

After both commands worked manually, I updated:

```text
/scripts/media_backup.sh
```

to contain:

```bash
#!/bin/bash

zip -r /backup/xfusioncorp_media.zip /var/www/html/media

scp /backup/xfusioncorp_media.zip ubuntu@172.31.18.142:/tmp/
```

This script now performs the whole backup process.

---

## Running the complete automation

I ran:

```bash
/scripts/media_backup.sh
```

The output showed:

```text
updating: var/www/html/media/
updating: var/www/html/media/image1.jpg
updating: var/www/html/media/image2.jpg
xfusioncorp_media.zip
```

This confirmed both operations completed:

```text
ZIP backup ✅
SCP transfer ✅
```

---

# Complete Flow

The final automation looks like this:

```text
/scripts/media_backup.sh
        ↓
Read /var/www/html/media
        ↓
Create ZIP archive
        ↓
/backup/xfusioncorp_media.zip
        ↓
Use SCP
        ↓
Authenticate with SSH key
        ↓
Backup Server
172.31.18.142
        ↓
/tmp/xfusioncorp_media.zip
```

---

# Understanding SSH Errors

One useful troubleshooting lesson from this task was the difference between SSH errors.

### Connection timeout

```text
Connection timed out
```

Usually points toward:

```text
Security Group
Firewall
Network route
Wrong IP
```

### Permission denied

```text
Permission denied (publickey)
```

Usually points toward:

```text
Wrong SSH user
Missing public key
Wrong private key
Incorrect authorized_keys
Key permissions
```

In this task, the error was:

```text
Permission denied (publickey)
```

So I knew the network was working and focused on authentication.

---

# Commands Used

Create source directory:

```bash
mkdir -p /var/www/html/media
```

Create sample files:

```bash
touch /var/www/html/media/image1.jpg /var/www/html/media/image2.jpg
```

Create backup directory:

```bash
mkdir -p /backup
```

Check ZIP:

```bash
zip -v
```

Create archive:

```bash
zip -r /backup/xfusioncorp_media.zip /var/www/html/media
```

Check archive:

```bash
ls -lh /backup/xfusioncorp_media.zip
```

List ZIP contents:

```bash
unzip -l /backup/xfusioncorp_media.zip
```

Create scripts directory:

```bash
mkdir -p /scripts
```

Create script:

```bash
nano /scripts/media_backup.sh
```

Make script executable:

```bash
chmod +x /scripts/media_backup.sh
```

Run script:

```bash
/scripts/media_backup.sh
```

Check SSH files:

```bash
ls -l /root/.ssh
```

View public key:

```bash
cat /root/.ssh/id_ed25519.pub
```

Test SSH:

```bash
ssh ubuntu@172.31.18.142
```

Copy archive:

```bash
scp /backup/xfusioncorp_media.zip ubuntu@172.31.18.142:/tmp/
```

Verify remote archive:

```bash
ls -lh /tmp/xfusioncorp_media.zip
```

---

# What I Learned

Today I learned how a Bash script can combine several commands into one reusable automation.

Instead of manually doing:

```text
Create archive
Copy archive
Verify
```

every time, I can put the commands into a script and run:

```bash
/scripts/media_backup.sh
```

I also learned that scripting is easier when I first test each command manually.

My process was:

```text
Test ZIP command manually
        ↓
Verify archive
        ↓
Test SSH manually
        ↓
Fix authentication
        ↓
Test SCP manually
        ↓
Verify remote copy
        ↓
Put commands into script
        ↓
Run full automation
```

This helped me troubleshoot one problem at a time.

---

# Main Takeaway

The biggest lesson from Day 10 was:

```text
Do not automate an untested process.
```

A better approach is:

```text
Understand task
      ↓
Run commands manually
      ↓
Verify each command
      ↓
Fix problems
      ↓
Put working commands into script
      ↓
Run automation
      ↓
Verify final result
```

I also combined several things I learned in earlier days:

```text
Linux files
+
permissions
+
SSH keys
+
SCP
+
Bash scripting
=
automated remote backup
```

This was my first step toward turning manual Linux administration into repeatable DevOps automation.