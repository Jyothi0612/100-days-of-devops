# Day 1 - Linux Service User with Non-Interactive Shell

## What was the task?

Create a Linux user called `backupsvc` and make sure that user cannot open an interactive terminal.

The idea is that `backupsvc` is meant for a service or application, not for a real person.

---

## What is a service user?

A service user is a Linux account used by an application or service.

For example, a backup service may need its own user so Linux can control which files it can access and which processes it can run.

Instead of running everything as `root`, we create a separate user with only the permissions the service actually needs.

This is safer because `root` has full control over the server.

---

## Why not run the backup service as root?

At first, I thought the main problem would be that root could get overloaded.

But I learned that the real problem is security.

`root` can read, modify, and delete almost anything on the system.

If a backup application running as root gets compromised, the attacker could potentially control the whole server.

Using a separate user like `backupsvc` limits what the service can do.

This follows the idea of least privilege: give a user or service only the permissions it actually needs.

---

## Why does the service user need a non-interactive shell?

`backupsvc` is not a human user.

It does not need to SSH into the server and type commands in a terminal.

It only needs an identity so the backup application can run under that user and access the files it is allowed to use.

Because of that, we give it a non-interactive shell.

This prevents someone from normally logging into the server as `backupsvc` and getting a command prompt.

---

## What is `nologin`?

`nologin` is a program that prevents a user from getting an interactive shell.

I first checked where it was installed:

```bash
command -v nologin
```

Output:

```text
/usr/sbin/nologin
```

`command -v` basically asks the shell:

> If I run this command, where will you find it?

So now I knew that the correct path on my server was:

```text
/usr/sbin/nologin
```

---

## Creating the user

I created the user with:

```bash
sudo useradd -s /usr/sbin/nologin backupsvc
```

What I understand from this command:

```text
sudo
```

Run the command with administrator privileges.

```text
useradd
```

Create a Linux user.

```text
-s
```

Set the shell for the user.

```text
/usr/sbin/nologin
```

Use `nologin` as the shell so the user cannot get an interactive terminal.

```text
backupsvc
```

The username I want to create.

---

## How did I verify it?

I used:

```bash
getent passwd backupsvc
```

Output:

```text
backupsvc:x:1001:1001::/home/backupsvc:/usr/sbin/nologin
```

`getent` means get an entry from a system database.

`passwd` here refers to the Linux user account database. It does not mean that the command is showing the user's password.

---

## Understanding the output

```text
backupsvc:x:1001:1001::/home/backupsvc:/usr/sbin/nologin
```

### `backupsvc`

This is the username.

### `x`

The password information is not stored directly here.

Linux normally stores password hashes in the protected `/etc/shadow` file.

### First `1001`

This is the UID, or User ID.

Linux internally identifies the user using this number.

### Second `1001`

This is the GID, or Group ID.

It identifies the user's primary group.

### Empty field `::`

This is the comment or user information field.

Nothing was configured here, so it is empty.

### `/home/backupsvc`

This is the home directory configured for the user.

It does not necessarily mean that the directory was created. It is the path stored in the user's account information.

### `/usr/sbin/nologin`

This is the user's shell.

This is the most important part of this task because it confirms that `backupsvc` does not have a normal interactive shell like `/bin/bash`.

---

## What I learned today

Today I learned that Linux users are not only for humans.

Applications and services can also have their own users.

I also learned why services should not automatically run as root. Giving every application root access creates unnecessary security risk.

A better approach is to create a separate service account and give it only the permissions it needs.

I learned that a normal human user may use a shell like:

```text
/bin/bash
```

while a service account that should not allow interactive login can use:

```text
/usr/sbin/nologin
```

I also learned how to check where a command exists using:

```bash
command -v
```

and how to check Linux user information using:

```bash
getent passwd <username>
```

The biggest thing I learned is that in DevOps I should not just run a command and assume it worked.

The better approach is:

```text
Understand the requirement
        ↓
Make the change
        ↓
Verify the result
```

## Commands used

```bash
command -v nologin
sudo useradd -s /usr/sbin/nologin backupsvc
getent passwd backupsvc
```
