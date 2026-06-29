---
title: Investigating Windows
summary: TryHackMe Windows forensics room investigating a previously compromised Windows Server host through RDP, account activity, scheduled tasks, logs, network artifacts, and web shell evidence.
date: 2026-06-29
difficulty: easy
os: Windows Server 2016
tags: [TryHackMe, Windows, Forensics, Incident Response, Event Logs, Scheduled Tasks]
url: https://tryhackme.com/room/investigatingwindows
---

# CTF Room: Investigating Windows
- [Link to room](https://tryhackme.com/room/investigatingwindows)
- **Difficulty:** Easy
- **Category:** Windows Forensics, Incident Response
- **OS:** Windows Server 2016

## 1. Brief
This room is focused on investigating a Windows machine that has previously been compromised.

The task is to connect to the target over RDP and collect evidence from the system, including account activity, scheduled tasks, network connections, password dumping artifacts, web shell traces, and DNS poisoning indicators.

## 2. Lab Setup
The TryHackMe AttackBox and target machine were deployed for the investigation.

### Machine Details
```bash
Attacker IP: 10.129.81.118
Target IP:   10.129.162.90
Target name: Windows Forensics-badr
```

### RDP Credentials
```bash
Username: Administrator
Password: letmein123!
```

The room notes that the machine does not respond to ICMP, so a failed ping does not mean the host is offline.

### RDP Connection
```bash
xfreerdp /v:10.129.162.90 /u:Administrator /p:'letmein123!' +clipboard /dynamic-resolution
```

## 3. Investigation Questions

#### What's the version and year of the Windows machine?
### Answer
```bash
Windows Server 2016
```

### Evidence
Checked the version through Windows Settings after connecting over RDP.

#### Which user logged in last?
### Answer
```bash
Administrator
```

### Evidence
```bash
Get-LocalUser | Select-Object Name, LastLogon
```

#### When did John log onto the system last?
### Answer
```bash
03/02/2019 5:48:32 PM
```

### Evidence
```bash
Get-LocalUser | Select-Object Name, LastLogon
```

#### What IP does the system connect to when it first starts?
### Answer
```bash
10.34.2.3
```

### Evidence
Found using Registry Editor from Run.

The `UpdateSvc` entry called out to this IP using `net user` through `p.exe`. This also briefly flashed up after connecting to the machine over RDP.

#### What two accounts had administrative privileges other than the Administrator user?
### Answer
```bash
Guest, Jenny
```

### Evidence
```bash
Get-LocalGroupMember -Group "Administrators"
```

#### What's the name of the scheduled task that is malicious?
### Answer
```bash
Clean file system
```

### Evidence
Found in Task Scheduler under the Active Tasks view. This task stood out as suspicious.

#### What file was the task trying to run daily?
### Answer
```bash
nc.ps1
```

### Evidence
Found under the Actions tab for the suspicious scheduled task in Task Scheduler.

#### What port did this file listen locally for?
### Answer
```bash
1348
```

### Evidence
```bash
$task = Get-ScheduledTask | Where-Object TaskName -EQ "Clean file system"
$task.Actions
```
The port was visible as a parameter in the scheduled task action.

#### When did Jenny last logon?
### Answer
```bash
Never
```

### Evidence
```bash
net user Jenny
```

#### At what date did the compromise take place?
### Answer
```bash
03/02/2019
```

### Evidence
Found in File Explorer by checking `C:\TMP` and reviewing when the PowerShell files were created.

#### During the compromise, at what time did Windows first assign special privileges to a new logon?
### Answer
```bash
03/02/2019 4:04:49 PM
```

### Evidence
Found in Event Viewer by filtering the Security event logs for special privilege assignment activity.

#### What tool was used to get Windows passwords?
### Answer
```bash
Mimikatz
```

### Evidence
Found `mim.exe` in `C:\TMP`. The `mim-out` file also contained the full name of the tool.

#### What was the attacker's external control and command servers IP?
### Answer
```bash
76.32.97.132
```

### Evidence
Found in:

```bash
C:\Windows\System32\drivers\etc\hosts
```

The hosts file redirected `google.com` to `76.32.97.132`, which is not a legitimate Google IP.

#### What was the extension name of the shell uploaded via the server's website?
### Answer
```bash
.jsp
```

### Evidence
Found suspicious `.jsp` files under:

```bash
C:\inetpub\wwwroot
```

#### What was the last port the attacker opened?
### Answer
```bash
1337
```

### Evidence
Checked Windows Firewall inbound rules, filtered by group, and reviewed the rules without a group. Ports `1337` and `8888` were present, with `1337` allowing outside connections for development.

#### Check for DNS poisoning, what site was targeted?
### Answer
```bash
google.com
```

### Evidence
The hosts file showed a malicious redirect for `google.com`, pointing it at the attacker's command and control IP.

## 4. Investigation Notes
Useful evidence sources for this room:

- Event Viewer logs, especially Security events for logon activity and special privilege assignment.
- Local Users and Groups for administrator group membership.
- Task Scheduler for malicious scheduled tasks.
- Startup folders, services, and registry run keys for persistence.
- Netstat or firewall rules for listening ports.
- IIS logs or web server directories for uploaded shell evidence.
- Hosts file and DNS settings for poisoning indicators.

## 5. Summary
This Windows Server 2016 machine had been compromised on `03/02/2019`. The attacker created or used accounts with administrative privileges, configured a suspicious scheduled task named `Clean file system`, used `nc.ps1` for local listening, and left password dumping evidence from Mimikatz in `C:\TMP`.

Additional evidence showed web shell activity through suspicious `.jsp` files in the IIS web root, a firewall rule exposing port `1337`, and DNS poisoning through the hosts file by redirecting `google.com` to the attacker's command and control IP.
