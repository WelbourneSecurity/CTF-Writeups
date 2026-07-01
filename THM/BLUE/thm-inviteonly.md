---
title: Invite Only
summary: TryHackMe SOC threat-intelligence room pivoting from a flagged IP and SHA256 hash to malware family, dropped files, phishing technique, and campaign reporting.
date: 2026-07-01
tags: [TryHackMe, SOC, Threat Intelligence, IOCs, Malware Analysis, AsyncRAT, Phishing]
difficulty: easy
os: Windows
url: https://tryhackme.com/room/invite-only
---

# CTF Room: Invite Only
- [Link to room](https://tryhackme.com/room/invite-only)
- [Badge earned: Lookup Champion](https://tryhackme.com/welbournesecurity/badges/lookup-champion)
- **Difficulty:** Easy
- **Category:** SOC, Threat Intelligence, Malware Triage
- **OS:** Windows

## 1. Brief
Invite Only is a TryHackMe threat-intelligence room. The scenario gives two escalated indicators from an L1 analyst and asks us to turn them into usable intelligence.

The starting indicators were:

```text
Flagged IP: 101[.]99[.]76[.]120
Flagged SHA256: ||5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f||
```

The room uses the TryDetectThis2.0 application to pivot between hashes, files, URLs, IPs, relations, dropped files, and public reporting.

## 2. Flagged Hash Analysis

#### What is the name of the file identified with the flagged SHA256 hash?
I started by searching the flagged SHA256 in TryDetectThis2.0. VirusTotal-style metadata also linked the hash to the same filename.

### Answer
```text
syshelpers.exe
```

#### What is the file type associated with the flagged SHA256 hash?
The file metadata showed it was a Windows executable.

### Answer
```text
Win32 EXE
```

#### What are the execution parents of the flagged hash?
Under `Relations`, the execution parents showed the process chain that led to the flagged hash.

### Answer
```text
361GJX7J,installer.exe
```

#### What is the name of the file being dropped?
The same relations view showed a dropped file associated with the flagged hash.

### Answer
```text
AClient.exe
```

## 3. Parent Hash Pivot

#### Research the second hash in question 3 and list the four malicious dropped files in the order they appear.
I pivoted into the second execution-parent hash from the previous question and reviewed its dropped files.

Only the malicious dropped files were needed.

### Answer
```text
searchhost.exe,syshelpers.exe,nat.vbs,runsys.vbs
```

## 4. Flagged IP Analysis

#### Analyse the files related to the flagged IP. What is the malware family that links these files?
I searched the flagged IP in TryDetectThis2.0 and reviewed related files and detections.

The common malware family linking the files was AsyncRAT.

### Answer
```text
AsyncRAT
```

## 5. Public Report Pivot

#### What is the title of the original report where these flagged indicators are mentioned?
The community/reporting context for the indicators pointed to a public writeup about hijacked Discord invites and multi-stage malware delivery.

### Answer
```text
From Trust to Threat: Hijacked Discord Invites Used for Multi-Stage Malware Delivery
```

#### Which tool did the attackers use to steal cookies from the Google Chrome browser?
The report described the use of ChromeKatz for browser cookie theft.

### Answer
```text
ChromeKatz
```

#### Which phishing technique did the attackers use?
The campaign used ClickFix-style social engineering.

### Answer
```text
ClickFix
```

#### What is the name of the platform that was used to redirect a user to malicious servers?
The report tied the redirection chain to Discord invite abuse.

### Answer
```text
DISCORD
```

## 6. Summary
The flagged SHA256 identified `syshelpers.exe`, a Win32 executable. Pivoting through execution parents and dropped files exposed the supporting files and follow-on payloads.

The flagged IP linked the activity to AsyncRAT. From there, the public reporting connected the campaign to hijacked Discord invites, ClickFix phishing, and ChromeKatz-based Chrome cookie theft.
