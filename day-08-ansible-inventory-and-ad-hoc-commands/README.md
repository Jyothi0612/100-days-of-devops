# Day 8 - Ansible Inventory and Ad-Hoc Commands

## What was the task?

The goal of Day 8 was to start using Ansible from my Ubuntu EC2 instance to manage my RHEL EC2 instance.

For this lab:

```text
Ubuntu EC2
Private IP: 172.31.28.111
Role: Ansible control node
```

```text
RHEL EC2
Private IP: 172.31.27.84
Role: Managed node
```

The main goal was to:

```text
Install/check Ansible
        ↓
Create an inventory
        ↓
Connect to the RHEL server
        ↓
Test Ansible communication
        ↓
Run a real command remotely
```

---

## What is Ansible?

Ansible is an automation and configuration management tool.

Without Ansible, I might SSH into each server manually and run the same command again and again.

Example:

```text
ssh server1
install package

ssh server2
install package

ssh server3
install package
```

That is okay for a few servers, but it becomes slow and error-prone when there are many servers.

With Ansible:

```text
One control node
      ↓
Ansible
      ↓
Multiple managed servers
```

I can run the same task against many servers from one place.

---

## Control Node and Managed Node

The machine where Ansible is installed is called the:

```text
control node
```

The servers Ansible manages are called:

```text
managed nodes
```

In my lab:

```text
Ubuntu EC2
172.31.28.111
     ↓
Control Node
     ↓
Ansible
     ↓
SSH
     ↓
RHEL EC2
172.31.27.84
     ↓
Managed Node
```

---

## Why Day 7 Was Important

In Day 7, I configured SSH key authentication.

That was useful because Ansible normally connects to managed nodes over SSH.

Without SSH keys:

```text
Ansible
   ↓
connect to server
   ↓
password prompt
   ↓
automation gets interrupted
```

With SSH keys:

```text
Ansible
   ↓
SSH key authentication
   ↓
server accepts connection
   ↓
automation continues
```

---

## Checking Ansible

On the Ubuntu control node, I checked the installed version using:

```bash
ansible --version
```

My output showed:

```text
ansible [core 2.20.1]
```

Some other useful details were:

```text
executable location = /usr/bin/ansible
python version = 3.14.4
config file = None
```

This told me:

```text
Ansible is installed
Python is available
No custom ansible.cfg is loaded
```

---

## Creating the Ansible Lab Directory

I created a working directory:

```bash
mkdir -p ~/ansible-lab
cd ~/ansible-lab
```

This is where I kept my inventory file.

---

## What is an Inventory?

An inventory tells Ansible which servers it should manage.

I created a file called:

```text
inventory
```

with:

```text
[rhel]
172.31.27.84 ansible_user=ec2-user
```

Breakdown:

```text
[rhel]
→ inventory group name
```

```text
172.31.27.84
→ RHEL server private IP
```

```text
ansible_user=ec2-user
→ SSH username Ansible should use
```

So the inventory basically tells Ansible:

> Manage this RHEL server and connect as `ec2-user`.

---

## First Ansible Ping Test

I tested the connection using:

```bash
ansible all -i inventory -m ping
```

Breakdown:

```text
ansible all
→ run against all hosts in the inventory
```

```text
-i inventory
→ use this inventory file
```

```text
-m ping
→ use Ansible's ping module
```

Important:

Ansible's `ping` module is not the same as normal network `ping`.

It checks whether Ansible can:

```text
SSH into the server
run Python
execute an Ansible module
return a result
```

---

## First Error: Host Key Verification Failed

My first attempt failed with:

```text
Host key verification failed
```

That meant:

```text
Network connection worked
        ↓
SSH reached the RHEL server
        ↓
But the host key was not trusted yet
```

So this was not a network timeout.

It was an SSH trust issue.

I then manually connected using:

```bash
ssh ec2-user@172.31.27.84
```

SSH asked:

```text
Are you sure you want to continue connecting?
```

I typed:

```text
yes
```

This added the RHEL server to:

```text
~/.ssh/known_hosts
```

---

## Second Error: Permission Denied

After accepting the host key, I got:

```text
Permission denied (publickey,...)
```

This told me the SSH connection reached the server, but authentication failed.

Then I noticed something important.

My shell prompt was:

```text
root@ip-172-31-28-111
```

So I was running Ansible as `root`.

But the SSH key I created on Day 7 belonged to:

```text
ubuntu
```

The private key was here:

```text
/home/ubuntu/.ssh/id_ed25519
```

while root normally looks in:

```text
/root/.ssh/
```

So root was not automatically using the correct SSH private key.

---

## Verifying the SSH Key

I checked that the key still existed:

```bash
ls -l /home/ubuntu/.ssh/id_ed25519
```

The output showed:

```text
-rw------- 1 ubuntu ubuntu ...
```

Then I manually tested the key:

```bash
ssh -i /home/ubuntu/.ssh/id_ed25519 ec2-user@172.31.27.84
```

The connection worked.

That proved:

```text
Network      ✓
SSH server   ✓
Public key   ✓
Private key  ✓
```

The only issue was that Ansible did not know which key to use.

---

## Updating the Inventory with the SSH Key

I updated the inventory to:

```text
[rhel]
172.31.27.84 ansible_user=ec2-user ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_ed25519
```

Now the inventory tells Ansible:

```text
Connect to 172.31.27.84
        ↓
Use user ec2-user
        ↓
Use private key:
        ↓
/home/ubuntu/.ssh/id_ed25519
```

---

## Successful Ansible Ping

I ran:

```bash
ansible all -i inventory -m ping
```

This time I got:

```text
172.31.27.84 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

That confirmed:

```text
Inventory found the server     ✓
SSH connection worked          ✓
Correct SSH key was used       ✓
Python existed on the RHEL host ✓
Ansible module executed        ✓
```

---

## Python Interpreter Warning

Ansible also showed a warning that it discovered:

```text
/usr/bin/python3.12
```

on the RHEL server.

This was not an error.

It just means Ansible automatically found Python on the managed node and used it.

---

## Running a Real Ad-Hoc Command

After the ping worked, I ran:

```bash
ansible rhel -i inventory -m command -a "hostname"
```

Breakdown:

```text
ansible rhel
→ run against the [rhel] group
```

```text
-i inventory
→ use the inventory file
```

```text
-m command
→ use Ansible's command module
```

```text
-a "hostname"
→ run the hostname command on the remote server
```

The output was:

```text
172.31.27.84 | CHANGED | rc=0 >>
ip-172-31-27-84.ec2.internal
```

---

## Understanding the Output

```text
172.31.27.84
→ the managed RHEL server
```

```text
rc=0
→ return code 0
→ command succeeded
```

```text
ip-172-31-27-84.ec2.internal
→ hostname returned by the remote server
```

I also learned that:

```text
CHANGED
```

does not always mean the hostname itself changed.

The `command` module reports that it ran a command, but Ansible does not always know whether that command changed the system.

---

## What I Learned

I learned the basic Ansible architecture:

```text
Control Node
      ↓
Inventory
      ↓
SSH
      ↓
Managed Node
```

I learned that the inventory tells Ansible:

```text
which server to manage
which user to use
which SSH key to use
```

I also learned that Ansible errors can tell me which layer is failing.

For example:

```text
Host key verification failed
→ SSH trust / known_hosts problem
```

```text
Permission denied (publickey)
→ SSH authentication problem
```

```text
SUCCESS + pong
→ Ansible communication is working
```

I also learned that Ansible's ping module checks more than basic network reachability.

It confirms:

```text
SSH works
Python works
Ansible can execute a module
```

---

## Commands Used

Check Ansible:

```bash
ansible --version
```

Create lab directory:

```bash
mkdir -p ~/ansible-lab
cd ~/ansible-lab
```

Inventory:

```text
[rhel]
172.31.27.84 ansible_user=ec2-user ansible_ssh_private_key_file=/home/ubuntu/.ssh/id_ed25519
```

Test Ansible:

```bash
ansible all -i inventory -m ping
```

Manual SSH test:

```bash
ssh ec2-user@172.31.27.84
```

Check the SSH private key:

```bash
ls -l /home/ubuntu/.ssh/id_ed25519
```

Test with the exact key:

```bash
ssh -i /home/ubuntu/.ssh/id_ed25519 ec2-user@172.31.27.84
```

Run a remote hostname command:

```bash
ansible rhel -i inventory -m command -a "hostname"
```

---

## Main Takeaway

The biggest lesson from Day 8 was that Ansible depends on several pieces working together:

```text
Inventory
   ↓
Network
   ↓
SSH trust
   ↓
SSH authentication
   ↓
Python
   ↓
Ansible module
```

When something fails, I should not randomly change things.

I should read the error and figure out which layer is broken.

My Day 8 workflow was:

```text
Check Ansible
      ↓
Create inventory
      ↓
Test connection
      ↓
Read the error
      ↓
Fix SSH trust
      ↓
Fix SSH key usage
      ↓
Get pong
      ↓
Run a real command
```