# Day 9 - MariaDB Troubleshooting

## What was the task?

The goal of Day 9 was to learn how to troubleshoot a database service when it fails.

Instead of randomly restarting services or reinstalling packages, I wanted to follow a proper troubleshooting process:

```text
Check service
     ↓
Read the error
     ↓
Find root cause
     ↓
Fix only the problem
     ↓
Start service
     ↓
Verify
```

For this lab, I used my Ubuntu 26.04 EC2 instance.

Because this was a clean EC2 server and MariaDB was not already broken, I installed MariaDB and then created a safe configuration failure myself so I could practice troubleshooting it.

---

## What is MariaDB?

MariaDB is a relational database server.

Applications can use it to store and retrieve data.

A simple flow looks like:

```text
Application
    ↓
MariaDB
    ↓
Database
    ↓
Data returned to application
```

If MariaDB is down:

```text
Application
    ↓
tries to connect
    ↓
MariaDB unavailable
    ↓
Database request fails
```

---

## First Troubleshooting Step

The first thing I learned is:

> Do not guess the problem.

Check the service first.

I ran:

```bash
sudo systemctl status mariadb
```

Initially I got:

```text
Unit mariadb.service could not be found.
```

This did not mean MariaDB was broken.

It meant systemd could not find a MariaDB service on that server.

So I needed more information.

---

## Checking the Operating System

I ran:

```bash
cat /etc/os-release
```

The server showed:

```text
Ubuntu 26.04 LTS
```

This confirmed I was working on Ubuntu.

---

## Checking Whether MariaDB Was Installed

I ran:

```bash
dpkg -l | grep -i mariadb
```

There was no output.

That confirmed MariaDB was not installed.

So the situation was:

```text
MariaDB service not found
        ↓
Check installed packages
        ↓
No MariaDB packages
        ↓
MariaDB not installed
```

---

## Installing MariaDB

I installed MariaDB using:

```bash
apt update
apt install mariadb-server -y
```

After installation, I checked:

```bash
systemctl status mariadb
```

The important output was:

```text
Active: active (running)
Status: "Taking your SQL requests now..."
```

The logs also showed:

```text
ready for connections
```

and:

```text
port: 3306
```

This confirmed MariaDB was running successfully.

---

## What is Port 3306?

MariaDB normally uses:

```text
3306
```

as its database port.

My output showed:

```text
Server socket created on IP: '127.0.0.1', port: '3306'
```

This meant MariaDB was listening locally on port 3306.

---

## Checking the Database Itself

A running systemd service does not always guarantee that the application itself is healthy.

So I ran:

```bash
mariadb-admin ping
```

The output was:

```text
mysqld is alive
```

This confirmed that MariaDB itself was responding.

So there are two different checks:

```text
systemctl status mariadb
→ Is the service process running?
```

```text
mariadb-admin ping
→ Is the database actually responding?
```

---

# Creating a Safe Failure

Because MariaDB was healthy, I intentionally created a safe configuration problem so I could practice troubleshooting.

Before doing that, I checked the MariaDB runtime directory:

```bash
ls -ld /run/mysqld
```

Output:

```text
drwxr-xr-x 2 mysql mysql ... /run/mysqld
```

The important part was:

```text
mysql mysql
```

This means:

```text
Owner → mysql
Group → mysql
```

MariaDB runs using the `mysql` user, so this directory ownership was correct.

---

## Stopping MariaDB

Before introducing the test failure, I stopped MariaDB:

```bash
systemctl stop mariadb
```

---

## Creating the Broken Configuration

I created this file:

```text
/etc/mysql/mariadb.conf.d/99-day9-broken.cnf
```

using:

```bash
printf '[mysqld]\ninvalid_option_for_day9=1\n' > /etc/mysql/mariadb.conf.d/99-day9-broken.cnf
```

The file contained:

```ini
[mysqld]
invalid_option_for_day9=1
```

This was intentionally invalid.

MariaDB does not understand:

```text
invalid_option_for_day9
```

The expected flow was:

```text
MariaDB starts
      ↓
reads configuration
      ↓
finds invalid option
      ↓
startup fails
```

---

## Trying to Start MariaDB

I ran:

```bash
systemctl start mariadb
```

It failed with:

```text
Job for mariadb.service failed because the control process exited with error code.
```

This message told me MariaDB failed, but it did not yet tell me the actual root cause.

That was an important lesson.

```text
Service failed
≠
Root cause found
```

---

## Reading the Detailed Error

I ran:

```bash
systemctl status mariadb --no-pager -l
```

The important error was:

```text
[ERROR] /usr/sbin/mariadbd: unknown variable 'invalid_option_for_day9=1'
```

Then:

```text
[ERROR] Aborting
```

Now I knew the actual problem.

MariaDB was failing because it found an unknown configuration variable.

The troubleshooting flow became:

```text
MariaDB failed
      ↓
Check status
      ↓
Read logs
      ↓
unknown variable
      ↓
Bad configuration
```

---

## What Do `--no-pager` and `-l` Mean?

I used:

```bash
systemctl status mariadb --no-pager -l
```

`--no-pager` means:

```text
print the output directly in the terminal
```

`-l` means:

```text
show full lines without cutting long messages
```

This made it easier to see the real error.

---

## Fixing the Problem

Because I knew exactly which file caused the problem, I removed only that file:

```bash
rm /etc/mysql/mariadb.conf.d/99-day9-broken.cnf
```

I did not reinstall MariaDB.

I did not delete database files.

I did not reboot the server.

I fixed only the actual root cause.

---

## Starting MariaDB Again

After removing the invalid configuration, I ran:

```bash
systemctl start mariadb
```

Then:

```bash
systemctl status mariadb --no-pager
```

The important output was:

```text
Active: active (running)
```

and:

```text
Status: "Taking your SQL requests now..."
```

The logs also showed:

```text
ready for connections
```

This confirmed MariaDB started successfully again.

---

## Final Database Verification

I ran:

```bash
mariadb-admin ping
```

and confirmed:

```text
mysqld is alive
```

So the complete verification was:

```text
MariaDB service running ✓
Database responding     ✓
```

---

# Understanding Different Errors

One of the biggest things I learned is that different errors point to different problems.

### Service not found

```text
Unit mariadb.service could not be found
```

Possible meaning:

```text
package not installed
wrong service name
wrong server
```

---

### Service failed

```text
Active: failed
```

This only tells me the service failed.

I still need to inspect the error.

---

### Configuration error

```text
unknown variable
```

This points toward:

```text
MariaDB configuration files
```

---

### Healthy service

```text
Active: active (running)
```

means the service is running.

---

### Healthy database

```text
mysqld is alive
```

means the database is responding.

---

## Commands Used

Check service:

```bash
systemctl status mariadb
```

Check OS:

```bash
cat /etc/os-release
```

Check installed packages:

```bash
dpkg -l | grep -i mariadb
```

Install MariaDB:

```bash
apt update
apt install mariadb-server -y
```

Check database:

```bash
mariadb-admin ping
```

Check runtime directory:

```bash
ls -ld /run/mysqld
```

Stop MariaDB:

```bash
systemctl stop mariadb
```

Create test failure:

```bash
printf '[mysqld]\ninvalid_option_for_day9=1\n' > /etc/mysql/mariadb.conf.d/99-day9-broken.cnf
```

Try starting MariaDB:

```bash
systemctl start mariadb
```

View detailed status:

```bash
systemctl status mariadb --no-pager -l
```

Remove bad configuration:

```bash
rm /etc/mysql/mariadb.conf.d/99-day9-broken.cnf
```

Start MariaDB again:

```bash
systemctl start mariadb
```

Verify service:

```bash
systemctl status mariadb --no-pager
```

Verify database:

```bash
mariadb-admin ping
```

---

## What I Learned

The biggest lesson from Day 9 was troubleshooting.

I learned that when a service fails, I should not immediately:

```text
reinstall packages
delete files
reboot the server
randomly change configuration
```

Instead, I should follow:

```text
Problem reported
      ↓
Check service status
      ↓
Read the exact error
      ↓
Understand the error
      ↓
Find root cause
      ↓
Fix only the root cause
      ↓
Restart/start service
      ↓
Verify service
      ↓
Verify application
```

I also learned that:

```bash
systemctl status
```

is one of the first places I should look when troubleshooting a Linux service.

The error message is often more useful than guessing.

---

## Main Takeaway

Day 9 taught me a mindset that I will use throughout DevOps:

```text
Observe
   ↓
Diagnose
   ↓
Fix
   ↓
Verify
```

A successful command is not enough.

I need to understand:

```text
Why did it fail?
What exactly did I change?
How do I know it is healthy again?
```

That is the main troubleshooting habit I learned today.