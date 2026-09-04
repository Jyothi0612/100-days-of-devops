# Day 4 - Linux Script Execution Permissions

## What was the task?

The goal was to understand Linux file permissions and make a shell script executable.

I learned that just because a script file exists, Linux does not automatically allow it to run.

A file needs execute permission:

```text
x = execute
```

---

## Linux file permissions

Linux permissions are shown like this:

```text
-rwxr-xr-x
```

They are split into four parts:

```text
- | rwx | r-x | r-x
    owner group others
```

The letters mean:

```text
r = read
w = write
x = execute
- = permission not given
```

The first character shows the file type:

```text
- = normal file
d = directory
l = symbolic link
```

---

## Owner, Group and Others

Linux checks who is trying to access the file.

```text
Owner  → user who owns the file
Group  → users belonging to the file's group
Others → everyone else
```

For example:

```text
-rw-r--r--
```

means:

```text
Owner  → read + write
Group  → read only
Others → read only
```

---

## Checking permissions

I used:

```bash
ls -l /etc/hosts
```

On my EC2 instance I got:

```text
-rw-r--r-- 1 root root 221 Jun 3 23:21 /etc/hosts
```

This showed me:

```text
Owner → root
Group → root
```

Since I was logged in as `ubuntu` and was not the owner or part of the root group, Linux used the `others` permissions for me.

So I could read `/etc/hosts`, but I could not normally modify it without sudo.

---

## Creating a test script

I created a script called:

```text
day4.sh
```

Initially its permissions were:

```text
-rw-rw-r--
```

There was no `x`, so the script was not executable.

When I tried:

```bash
./day4.sh
```

I got:

```text
Permission denied
```

This showed me that the execute permission was missing.

---

## Adding execute permission

I used:

```bash
chmod u+x day4.sh
```

What this means:

```text
chmod → change file permissions
u     → user / owner
+x    → add execute permission
```

After that:

```bash
ls -l day4.sh
```

showed:

```text
-rwxrw-r--
```

The important change was:

```text
rw- → rwx
       ↑
       execute permission added
```

---

## Shebang

My script contained:

```bash
#!/bin/bash
echo "Hello from Day 4"
```

The first line:

```bash
#!/bin/bash
```

is called a shebang.

```text
#!        → special marker
/bin/bash → use Bash to run this script
```

So when I run:

```bash
./day4.sh
```

Linux knows that Bash should execute the script.

The output was:

```text
Hello from Day 4
```

---

## Local computer vs EC2

I also noticed that file ownership depends on which machine I am using.

On my EC2 instance, a file created by the Ubuntu user may show:

```text
ubuntu ubuntu
```

On my Mac, I saw:

```text
jyothi staff
```

The permission concept is the same, but the owner and group depend on the machine and user that created the file.

---

## What did I learn?

I learned that Linux separates permissions into:

```text
read
write
execute
```

and applies them separately to:

```text
owner
group
others
```

I also learned that a shell script needs execute permission if I want to run it directly using:

```bash
./script.sh
```

The biggest thing I learned today was:

```text
File exists
    ↓
Check permissions
    ↓
No x
    ↓
Permission denied
    ↓
Add execute permission
    ↓
Script can run
```

---

## Commands used

```bash
ls -l /etc/hosts

printf '#!/bin/bash\necho "Hello from Day 4"\n' > day4.sh

cat day4.sh

ls -l day4.sh

./day4.sh

chmod u+x day4.sh

ls -l day4.sh

./day4.sh
```

## Main takeaway

Before running a script, I should understand who owns the file and which read, write, and execute permissions are given to the owner, group, and others.
