# Day 5 - SELinux Installation and Configuration

## What was the task?

The goal of Day 5 was to understand SELinux, check its current state, learn the difference between Enforcing and Permissive modes, and configure the RHEL server so SELinux will be fully disabled after the next reboot.

For this lab, I created a separate RHEL EC2 instance because my Ubuntu EC2 instance uses AppArmor by default.

---

## What is SELinux?

SELinux stands for:

```text
Security-Enhanced Linux
```

It adds another security layer on top of normal Linux permissions.

Normally Linux checks permissions like:

```text
read
write
execute
```

For example:

```text
Application
    ↓
Linux permissions
    ↓
Allow / Deny
```

With SELinux enabled, there is another check:

```text
Application
    ↓
Linux permissions
    ↓
SELinux policy
    ↓
Allow / Deny
```

This means even if normal Linux permissions allow access, SELinux can still block it.

Example:

```text
Linux permissions → ALLOW
SELinux policy    → DENY
                     ↓
Final result      → BLOCKED
```

Both security layers need to allow the action.

---

## Why is SELinux useful?

Imagine a web server gets compromised.

Without additional restrictions, the compromised application might try to access files or resources that it should not use.

SELinux can restrict what that application is allowed to do.

```text
Attacker compromises application
            ↓
Application tries accessing restricted resource
            ↓
SELinux checks policy
            ↓
Not allowed
            ↓
BLOCKED
```

This helps reduce the damage that a compromised application can cause.

---

## SELinux Modes

I learned about three SELinux states/modes.

### Enforcing

SELinux actively enforces its policies.

```text
Policy violation
      ↓
Blocked
      +
Logged
```

This is normally the secure mode used on production systems.

---

### Permissive

SELinux still checks policies, but it does not block violations.

```text
Policy violation
      ↓
Allowed
      +
Logged
```

Permissive mode is useful for testing and troubleshooting.

It lets us see what SELinux would have blocked without actually blocking the application.

---

### Disabled

SELinux is not enforcing a security policy.

```text
SELinux policy enforcement
          ↓
          OFF
```

---

## Checking SELinux Status

I connected to my RHEL EC2 instance and ran:

```bash
sestatus
```

The output was:

```text
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33
```

The most important lines were:

```text
SELinux status: enabled
Current mode: enforcing
Mode from config file: enforcing
```

This confirmed that SELinux was installed, enabled, and actively enforcing policies.

---

## `sestatus` vs `getenforce`

I also learned a quicker way to check the current SELinux mode.

```bash
getenforce
```

Output:

```text
Enforcing
```

The difference is:

```text
sestatus
→ shows detailed SELinux information

getenforce
→ shows only the current SELinux mode
```

So if I only want to quickly check the mode, I can use:

```bash
getenforce
```

---

## Temporarily Changing SELinux Mode

I changed SELinux from Enforcing to Permissive using:

```bash
sudo setenforce 0
```

Then I checked:

```bash
getenforce
```

Output:

```text
Permissive
```

So:

```text
setenforce 0
→ switch to Permissive
```

I changed it back to Enforcing using:

```bash
sudo setenforce 1
```

Then:

```bash
getenforce
```

returned:

```text
Enforcing
```

So:

```text
setenforce 1
→ switch to Enforcing
```

---

## Important: `setenforce` is temporary

One important thing I learned is that:

```bash
setenforce
```

only changes the current running SELinux mode.

It does not permanently change what happens after reboot.

```text
setenforce 0
        ↓
Permissive right now
        ↓
Server reboot
        ↓
Persistent configuration is checked again
```

So I need to understand the difference between:

```text
current state
```

and:

```text
state after reboot
```

---

## SELinux Persistent Configuration

I checked the SELinux configuration file using:

```bash
cat /etc/selinux/config
```

The important lines were:

```text
SELINUX=enforcing
SELINUXTYPE=targeted
```

### `SELINUX=enforcing`

This means the system is configured to use Enforcing mode.

### `SELINUXTYPE=targeted`

This means the targeted SELinux policy is being used.

The targeted policy mainly protects specific services and processes with SELinux rules.

---

## Current Mode vs Persistent Configuration

This was one of the main things I learned.

```text
getenforce
→ what SELinux is doing right now
```

```text
/etc/selinux/config
→ persistent SELinux configuration
```

And:

```text
setenforce
→ temporary runtime change
```

---

## RHEL 10 and Fully Disabling SELinux

I am using RHEL 10.

For fully disabling SELinux at boot, I used the kernel parameter:

```text
selinux=0
```

Before changing anything, I checked whether `grubby` was installed.

```bash
rpm -q grubby
```

`rpm -q` means:

```text
rpm     → RPM package database/tool
-q      → query
grubby  → package being checked
```

This basically asks:

```text
Is grubby installed?
```

---

## What is `grubby`?

`grubby` is used to manage kernel boot configuration.

The server reads kernel boot arguments when it starts.

So if we add:

```text
selinux=0
```

to the kernel arguments, SELinux will be disabled when the system boots.

---

## Configuring SELinux to be Disabled After Reboot

I ran:

```bash
sudo grubby --update-kernel ALL --args selinux=0
```

Breakdown:

```text
grubby
→ manage kernel boot settings
```

```text
--update-kernel ALL
→ apply the change to all installed kernels
```

```text
--args selinux=0
→ add selinux=0 to the kernel boot arguments
```

So the command basically means:

```text
For all installed kernels,
add selinux=0 when the system boots.
```

---

## Important: SELinux Was Not Disabled Immediately

After running the `grubby` command, SELinux was still Enforcing.

That is expected.

```text
Current session
→ Enforcing
```

```text
Next reboot
→ SELinux disabled
```

The command changes the boot configuration.

It does not immediately disable SELinux on the currently running system.

---

## Verifying the Kernel Boot Argument

I verified the change using:

```bash
sudo grubby --info=ALL | grep "args="
```

The output included:

```text
args="console=tty0 console=ttyS0,115200n8 nvme_core.io_timeout=4294967295 crashkernel=2G-64G:256M,64G-:512M $tuned_params selinux=0"
```

The important part was:

```text
selinux=0
```

This confirmed that the boot argument was successfully added.

---

## Final Verification

I checked the current SELinux mode again:

```bash
getenforce
```

It still returned:

```text
Enforcing
```

So at the end of the task:

```text
Current session → Enforcing
Next reboot     → SELinux disabled
```

I did not reboot the server because the task was specifically about configuring the next boot.

---

## Commands Used

```bash
sestatus
```

```bash
getenforce
```

```bash
sudo setenforce 0
```

```bash
getenforce
```

```bash
sudo setenforce 1
```

```bash
getenforce
```

```bash
cat /etc/selinux/config
```

```bash
rpm -q grubby
```

```bash
sudo grubby --update-kernel ALL --args selinux=0
```

```bash
sudo grubby --info=ALL | grep "args="
```

```bash
getenforce
```

---

## What I Learned

Today I learned that SELinux is an additional security layer on top of normal Linux permissions.

Normal permissions may allow an action, but SELinux can still block it based on its security policy.

I learned the difference between:

```text
Enforcing
Permissive
Disabled
```

I also learned that:

```text
Enforcing
→ block violations and log them

Permissive
→ allow violations but log them

Disabled
→ SELinux policy enforcement is off
```

I learned the difference between these commands:

```text
sestatus
→ detailed SELinux information

getenforce
→ current SELinux mode

setenforce
→ temporarily change the current mode
```

I also learned that:

```text
/etc/selinux/config
```

contains persistent SELinux configuration.

Another important thing I learned is the difference between the current running system and the next boot.

```text
Current system state
        ↓
getenforce
```

```text
Future boot configuration
        ↓
kernel arguments
        ↓
grubby
```

The biggest lesson from this task was:

```text
Check current state
        ↓
Understand the configuration
        ↓
Make the change
        ↓
Verify the change
        ↓
Understand when the change takes effect
```

In DevOps, I should not just run a command and assume everything is done.

I should always know whether a change happens immediately or only after a service restart or system reboot.

---

## Main Takeaway

SELinux provides an extra security layer on Linux systems.

For this lab, I learned how to inspect SELinux, temporarily switch between Enforcing and Permissive modes, understand its persistent configuration, and configure RHEL 10 so SELinux will be disabled on the next reboot.

My Day 5 workflow was:

```text
Understand SELinux
        ↓
Check current status
        ↓
Learn Enforcing vs Permissive
        ↓
Test temporary mode changes
        ↓
Check persistent configuration
        ↓
Configure next boot
        ↓
Verify kernel arguments
        ↓
Verify current state
```