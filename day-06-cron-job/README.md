# Day 6 - Create a Cron Job

## What was the task?

The goal was to create a cron job for the `root` user that runs every 5 minutes and writes:

```text
hello
```

into:

```text
/tmp/cron_text
```

The main thing I wanted to understand was how Linux can run commands automatically on a schedule.

---

## What is Cron?

Cron is a scheduler in Linux.

It lets me define:

```text
what command should run
        +
when it should run
```

Instead of manually running a command again and again, cron can do it automatically.

For example:

```text
Every 5 minutes
      ↓
Run a command
      ↓
Linux handles it automatically
```

This is useful for things like:

```text
backups
cleanup jobs
health checks
reports
scheduled scripts
```

---

## Cron Schedule Format

A cron schedule has 5 time fields:

```text
* * * * *
│ │ │ │ │
│ │ │ │ └── day of week
│ │ │ └──── month
│ │ └────── day of month
│ └──────── hour
└────────── minute
```

So the order is:

```text
minute
hour
day of month
month
day of week
```

---

## What does `*` mean?

`*` means:

```text
every possible value
```

For example:

```text
* * * * *
```

means:

```text
every minute
of every hour
of every day
of every month
on every day of the week
```

So this runs every minute.

---

## What does `*/5` mean?

`*/5` means:

```text
every 5 units
```

If it is in the minute field:

```text
*/5 * * * *
```

it means:

```text
every 5 minutes
```

Example run times:

```text
10:00
10:05
10:10
10:15
10:20
...
```

---

## The Actual Cron Job

The task required:

```text
Every 5 minutes
      ↓
Run:
echo hello > /tmp/cron_text
```

So the cron entry was:

```bash
*/5 * * * * echo hello > /tmp/cron_text
```

Breakdown:

```text
*/5
→ every 5 minutes
```

```text
* * * *
→ every hour/day/month/weekday
```

```text
echo hello
→ print the word hello
```

```text
>
→ write the output into a file
```

```text
/tmp/cron_text
→ file where the output is stored
```

---

## Creating the Root Cron Job

I opened root's crontab using:

```bash
sudo crontab -e
```

Because it was the first time, Linux showed:

```text
no crontab for root - using an empty one
```

It also asked me to choose an editor.

I selected:

```text
nano
```

because it was the easiest editor for me.

Then I added:

```bash
*/5 * * * * echo hello > /tmp/cron_text
```

---

## Saving the Crontab

In nano, I saved using:

```text
Ctrl + O
```

Then:

```text
Enter
```

and exited using:

```text
Ctrl + X
```

After saving, Linux showed:

```text
crontab: installing new crontab
```

That confirmed the cron entry was installed.

---

## Verifying the Crontab

I checked root's crontab using:

```bash
crontab -l
```

The important line was:

```text
*/5 * * * * echo hello > /tmp/cron_text
```

This confirmed the cron job was saved correctly.

---

## Checking the Cron Service

I also checked whether the cron service was running:

```bash
systemctl is-active cron
```

Output:

```text
active
```

This confirmed that the cron service was running.

---

## Checking the Server Time

I ran:

```bash
date
```

The server time was:

```text
Sun Sep 6 21:41:23 UTC 2026
```

Because the job runs every 5 minutes, the next run should happen around:

```text
21:45
21:50
21:55
22:00
```

---

## Verifying That the Cron Job Actually Ran

After waiting for the next 5-minute interval, I checked:

```bash
cat /tmp/cron_text
```

The output was:

```text
hello
```

That confirmed the cron job actually executed.

This was important because just seeing the cron entry in `crontab -l` does not prove the job really ran.

---

## What I Learned

I learned that cron is used to automate commands on a schedule.

I also learned how the 5 cron time fields work:

```text
minute
hour
day of month
month
day of week
```

I learned that:

```text
*
→ every value
```

and:

```text
*/5
→ every 5 units
```

I also learned the difference between:

```text
creating a cron job
```

and:

```text
verifying that the job actually ran
```

The commands I now understand are:

```text
crontab -e
→ edit cron jobs
```

```text
crontab -l
→ list cron jobs
```

```text
systemctl is-active cron
→ check whether cron service is running
```

```text
date
→ check server time
```

```text
cat /tmp/cron_text
→ verify the output created by the cron job
```

---

## Commands Used

```bash
sudo crontab -e
```

```bash
crontab -l
```

```bash
systemctl is-active cron
```

```bash
date
```

```bash
cat /tmp/cron_text
```

Cron entry:

```bash
*/5 * * * * echo hello > /tmp/cron_text
```

---

## Main Takeaway

The biggest thing I learned today is that automation is not complete until I verify it.

My workflow was:

```text
Understand the schedule
        ↓
Create the cron job
        ↓
Save the crontab
        ↓
Check the cron service
        ↓
Wait for the scheduled time
        ↓
Verify the job actually ran
```