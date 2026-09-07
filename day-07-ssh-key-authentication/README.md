# Day 7 - SSH Key Authentication

## What was the task?

The goal of Day 7 was to configure passwordless SSH between two Linux servers using SSH keys.

For this lab, I used:

```text
Ubuntu EC2
Private IP: 172.31.28.111
```

as the source server.

And:

```text
RHEL EC2
Private IP: 172.31.27.84
```

as the destination server.

The final goal was to connect from Ubuntu to RHEL like this:

```bash
ssh ec2-user@172.31.27.84
```

without using a password and without passing a `.pem` file manually.

---

## What is SSH key authentication?

SSH key authentication lets one machine prove its identity to another machine using a pair of cryptographic keys.

There are two keys:

```text
Private key
Public key
```

The idea is:

```text
Source server
     ↓
Private key stays here
     ↓
Public key is copied
     ↓
Destination server
```

The private key must stay secret.

The public key can safely be copied to the destination server.

---

## Private Key vs Public Key

### Private key

The private key stays on the machine that starts the SSH connection.

Example:

```text
~/.ssh/id_ed25519
```

This key should never be shared.

---

### Public key

The public key can be copied to another server.

Example:

```text
~/.ssh/id_ed25519.pub
```

The destination server uses this public key to verify that the connecting machine owns the matching private key.

---

## How SSH Key Authentication Works

The flow looks like this:

```text
Ubuntu EC2
Private key
     ↓
SSH connection
     ↓
RHEL EC2
authorized_keys
     ↓
Matching public key found
     ↓
Login allowed
```

So the server does not need the private key.

It only needs the matching public key.

---

## Generating the SSH Key Pair

On the Ubuntu EC2 instance, I generated a new SSH key pair using:

```bash
ssh-keygen -t ed25519
```

I accepted the default location:

```text
/home/ubuntu/.ssh/id_ed25519
```

This created two files:

```text
id_ed25519
id_ed25519.pub
```

The important difference is:

```text
id_ed25519
→ private key
→ keep secret
```

```text
id_ed25519.pub
→ public key
→ safe to copy
```

---

## Viewing the Public Key

My public key looked like:

```text
ssh-ed25519 AAAA... ubuntu@ip-172-31-28-111
```

It has three parts:

```text
ssh-ed25519
→ key type
```

```text
AAAA...
→ actual public key data
```

```text
ubuntu@ip-172-31-28-111
→ comment showing where the key was created
```

---

## What is `authorized_keys`?

On the destination server, SSH checks this file:

```text
~/.ssh/authorized_keys
```

For the RHEL `ec2-user`, that means:

```text
/home/ec2-user/.ssh/authorized_keys
```

This file contains public keys that are allowed to log in as that user.

The idea is:

```text
SSH client presents proof
        ↓
Server checks authorized_keys
        ↓
Matching public key exists
        ↓
Login allowed
```

---

## Preparing the Destination Server

On the RHEL server, I created the SSH directory and authorized keys file:

```bash
mkdir -p ~/.ssh
touch ~/.ssh/authorized_keys
```

Then I added the Ubuntu server's public key into:

```text
~/.ssh/authorized_keys
```

Important:

```text
authorized_keys stores public keys
```

It should not contain private keys.

---

## AWS Security Group Problem

At first, SSH from Ubuntu to RHEL did not work.

I tried:

```bash
ssh -o ConnectTimeout=5 ec2-user@172.31.27.84
```

and got:

```text
ssh: connect to host 172.31.27.84 port 22: Connection timed out
```

This told me something important.

A timeout means SSH did not even reach the destination server.

So this was not an SSH key problem yet.

It was a networking/security group problem.

---

## Fixing the Security Group

The RHEL security group needed to allow SSH from the Ubuntu EC2 private IP.

The Ubuntu private IP was:

```text
172.31.28.111
```

So I added this inbound rule:

```text
Type: SSH
Port: 22
Source: 172.31.28.111/32
```

The `/32` means:

```text
allow only this one IP address
```

After fixing the rule, SSH was able to reach the RHEL server.

---

## Final SSH Test

From the Ubuntu EC2 server, I ran:

```bash
ssh ec2-user@172.31.27.84
```

The first time, SSH asked:

```text
Are you sure you want to continue connecting?
```

I entered:

```text
yes
```

After that, the connection worked.

I did not use:

```bash
-i sample.pem
```

and I did not enter a password.

That confirmed SSH key authentication was working.

---

## Final Flow

The complete flow was:

```text
Ubuntu EC2
172.31.28.111
     ↓
Private key:
~/.ssh/id_ed25519
     ↓
SSH
     ↓
RHEL EC2
172.31.27.84
     ↓
Public key stored in:
~/.ssh/authorized_keys
     ↓
Login allowed
```

---

## What I Learned

I learned that SSH key authentication uses two keys:

```text
Private key
Public key
```

The private key stays on the source machine.

The public key is copied to the destination machine.

I also learned that:

```text
~/.ssh/authorized_keys
```

contains the public keys that are allowed to log in.

Another important thing I learned is how to tell the difference between a network issue and an SSH authentication issue.

If I see:

```text
Connection timed out
```

it usually means:

```text
network
security group
firewall
routing
port 22
```

If I see:

```text
Permission denied (publickey)
```

then the connection reached the server, but authentication failed.

That means I should check:

```text
SSH keys
authorized_keys
permissions
username
```

---

## Commands Used

Generate SSH keys:

```bash
ssh-keygen -t ed25519
```

Check SSH key files:

```bash
ls -l ~/.ssh/id_ed25519*
```

Prepare authorized keys on RHEL:

```bash
mkdir -p ~/.ssh
touch ~/.ssh/authorized_keys
```

Edit authorized keys:

```bash
nano ~/.ssh/authorized_keys
```

Test SSH with timeout:

```bash
ssh -o ConnectTimeout=5 ec2-user@172.31.27.84
```

Final connection:

```bash
ssh ec2-user@172.31.27.84
```

---

## Main Takeaway

The biggest lesson from Day 7 was that SSH key authentication is not just about generating keys.

The full setup involves:

```text
Generate key pair
      ↓
Keep private key safe
      ↓
Copy public key
      ↓
Add it to authorized_keys
      ↓
Allow network access
      ↓
Test SSH
      ↓
Verify no password is needed
```

I also learned that when SSH fails, the exact error message matters.

```text
Timeout
→ network problem
```

```text
Permission denied
→ authentication problem
```

That makes troubleshooting much easier.