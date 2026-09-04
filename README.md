# Day 2 - Temporary Linux User with Account Expiry

## What was the task?

Create a temporary Linux user named `tempdev` and make the account expire on:

```text
2026-09-15
```

The goal was to make sure the user can only use the account for a limited period.

---

## Why do we need temporary users?

Sometimes developers, contractors, or consultants only need access to a server for a few days or weeks.

We could create a normal user and delete it later, but as humans we may forget.

If we set an expiry date, Linux can automatically block the account after that date.

```text
Create user
   ↓
Set expiry date
   ↓
User works before expiry
   ↓
Expiry date arrives
   ↓
Account access is blocked
```

This reduces unnecessary access and improves security.

---

## Does an expired account get deleted?

No.

Account expiry does not delete the user.

The user still exists in Linux, and its files may still exist.

The main difference is that the account can no longer be normally used after the expiry date.

```text
Expired account ≠ Deleted account
```

---

## Creating the user

Since I was already logged in as `root`, I did not need to use `sudo`.

I created the user with:

```bash
useradd tempdev
```

`useradd` creates a new Linux user.

---

## Setting the expiry date

I used:

```bash
chage -E 2026-09-15 tempdev
```

What each part means:

```text
chage
```

Used to manage password and account aging settings.

```text
-E
```

Sets the account expiry date.

```text
2026-09-15
```

The date when the account should expire.

```text
tempdev
```

The user whose expiry date I am changing.

So the command basically means:

> Set the `tempdev` account to expire on September 15, 2026.

---

## How did I verify it?

I used:

```bash
chage -l tempdev
```

The `-l` option lists the current account aging information.

My output was:

```text
Last password change                                    : Sep 03, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Sep 15, 2026
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

The most important line for this task was:

```text
Account expires : Sep 15, 2026
```

That confirmed the requirement was successfully completed.

---

## What did I learn?

I learned that Linux accounts can have an expiry date.

This is useful when someone only needs server access temporarily.

Instead of depending on someone to remember to remove the account later, Linux can automatically block access after the configured date.

I also learned that account expiry and password expiry are different things.

In my output:

```text
Password expires : never
Account expires  : Sep 15, 2026
```

The password itself does not expire, but the entire account becomes unavailable after September 15.

I also learned the difference between these commands:

```bash
chage -E
```

Sets the account expiry date.

```bash
chage -l
```

Shows the current account aging and expiry information.

---

## Commands used

```bash
useradd tempdev
chage -E 2026-09-15 tempdev
chage -l tempdev
```

---

## Main takeaway

For temporary access, it is better to set an expiry date instead of relying on someone to remember to remove the user later.

My workflow for this task was:

```text
Understand the requirement
        ↓
Create the user
        ↓
Set the expiry date
        ↓
Verify the result
```
