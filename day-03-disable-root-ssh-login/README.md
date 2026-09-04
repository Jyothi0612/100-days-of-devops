# Day 3 - Disable Direct Root SSH Login

## What was the task?

The goal was to make the server more secure by stopping users from logging in directly as `root` over SSH.

`root` is the most powerful account on Linux, so allowing direct SSH access to it is risky.

---

## Why is direct root SSH login risky?

If someone manages to log in directly as `root`, they immediately get full control of the server.

`root` can:

* read or delete any file
* stop services
* change system settings
* create or remove users
* basically control the whole machine

A safer approach is:

```text
SSH as normal user
      ↓
use sudo when admin access is needed
```

This adds another layer of protection.

---

## What does `PermitRootLogin` mean?

SSH has a setting called:

```text
PermitRootLogin
```

If it is set to:

```text
PermitRootLogin no
```

then direct SSH login as `root` is blocked.

This does not delete the root user.

It only stops this:

```text
ssh root@server
```

---

## Checking the current setting

I searched the SSH config using:

```bash
grep -R "^[#[:space:]]*PermitRootLogin" /etc/ssh/sshd_config /etc/ssh/sshd_config.d/ 2>/dev/null
```

The result was:

```text
/etc/ssh/sshd_config:#PermitRootLogin prohibit-password
```

The `#` means the line is commented out.

I also learned that I do not need to memorize that whole `grep` command right now.

The simpler idea is:

```bash
grep PermitRootLogin /etc/ssh/sshd_config
```

`grep` searches for text inside a file.

---

## Checking that another admin user exists

Before blocking root SSH, I checked whether the `ubuntu` user had sudo access.

```bash
getent group sudo
```

Output:

```text
sudo:x:27:ubuntu
```

Then I checked its sudo permissions:

```bash
sudo -l -U ubuntu
```

This confirmed that `ubuntu` could run admin commands with sudo.

This was important because I did not want to lock myself out of the server.

---

## Backing up the SSH config

Before changing SSH settings, I created a backup:

```bash
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

This gives me a copy of the original config in case I need to restore it.

---

## Disabling root SSH login

I created a separate config file:

```bash
echo "PermitRootLogin no" > /etc/ssh/sshd_config.d/99-disable-root-login.conf
```

This adds:

```text
PermitRootLogin no
```

to the SSH configuration.

I used a separate file in `sshd_config.d` instead of editing the main file directly.

---

## Validating the config

Before reloading SSH, I checked for syntax errors:

```bash
sshd -t
```

It returned no output.

That means the SSH config was valid.

---

## Reloading SSH

I applied the change using:

```bash
systemctl reload ssh
```

This reloads the SSH service without fully stopping it.

---

## Verifying the effective setting

I checked what SSH was actually using:

```bash
sshd -T | grep permitrootlogin
```

Output:

```text
permitrootlogin no
```

This confirmed that the setting was active.

---

## Testing from my computer

I connected to the EC2 instance as the normal user:

```bash
ssh -i ~/Downloads/sample.pem ubuntu@ec2-50-19-174-5.compute-1.amazonaws.com
```

The login worked.

Then I tested root:

```bash
ssh -i ~/Downloads/sample.pem root@ec2-50-19-174-5.compute-1.amazonaws.com
```

Output:

```text
Permission denied (publickey).
```

That confirmed root SSH login was blocked.

---

## What did I learn?

I learned why direct root SSH access is risky.

I also learned that SSH settings are controlled through:

```text
/etc/ssh/sshd_config
```

and also through files inside:

```text
/etc/ssh/sshd_config.d/
```

I learned that before changing SSH settings, it is important to:

```text
check another admin user
      ↓
backup the config
      ↓
make the change
      ↓
validate the config
      ↓
reload SSH
      ↓
verify the effective setting
      ↓
test the login for real
```

The biggest lesson was that changing a config file is not enough.

I should always verify that the service is actually using the setting.

---

## Commands used

```bash
grep -R "^[#[:space:]]*PermitRootLogin" /etc/ssh/sshd_config /etc/ssh/sshd_config.d/ 2>/dev/null

getent group sudo

sudo -l -U ubuntu

cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

echo "PermitRootLogin no" > /etc/ssh/sshd_config.d/99-disable-root-login.conf

sshd -t

systemctl reload ssh

sshd -T | grep permitrootlogin

ssh -i ~/Downloads/sample.pem ubuntu@ec2-50-19-174-5.compute-1.amazonaws.com

ssh -i ~/Downloads/sample.pem root@ec2-50-19-174-5.compute-1.amazonaws.com
```
