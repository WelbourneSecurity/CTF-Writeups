---
title: PS Eclipse
summary: TryHackMe Splunk investigation room tracing a suspected ransomware incident through PowerShell download activity, scheduled task privilege escalation, C2 callbacks, and BlackSun ransomware IOCs.
date: 2026-07-02
tags: [TryHackMe, SOC, Splunk, PowerShell, Ransomware, Windows, Incident Response]
difficulty: medium
os: Windows
url: https://tryhackme.com/room/posheclipse
---

# CTF Room: PS Eclipse
- [Link to room](https://tryhackme.com/room/posheclipse)
- **Difficulty:** Medium
- **Category:** SOC, Splunk, Ransomware Investigation
- **OS:** Windows

## 1. Brief
PS Eclipse is a TryHackMe SOC room using Splunk to investigate suspected ransomware activity on Keegan's machine.

The customer reported that the machine still worked, but files had odd extensions. The job was to check Splunk events from Monday, May 16, 2022, and work out whether ransomware activity had taken place.

Useful pivots:

- Sysmon Event ID 1 for process creation
- Sysmon Event ID 3 for network connections
- Sysmon Event ID 11 for file creation
- Sysmon Event ID 22 for DNS queries
- PowerShell encoded command lines
- VirusTotal enrichment for script identity

## 2. Suspicious Binary

#### A suspicious binary was downloaded to the endpoint. What was the name of the binary?
I started with network-related Sysmon events. The noisy image path stood out quickly when grouped by process image.

### SPL
```spl
index=* EventCode=3 ComputerName="DESKTOP-TBV8NEF"
| table UtcTime, Image, DestinationPort, DestinationIp, SourcePort, SourceIp
| stats count by Image
| sort - count
```

Most of the hits came from a suspicious executable in `C:\Windows\Temp`.

### Answer
```text
||OUTSTANDING_GUTTER.exe||
```

#### What is the address the binary was downloaded from?
From there I pivoted into PowerShell. The suspicious process creation event used `-exec bypass` and `-enc`, so the interesting bit was the encoded payload.

### SPL
```spl
index=* ComputerName="DESKTOP-TBV8NEF" EventCode=1 Description="Windows PowerShell"
| table CommandLine
```

Decoding the Base64 revealed a payload that disabled Defender real-time monitoring, downloaded the binary, created a scheduled task, and ran it.

### Answer
```text
||http://886e-181-215-214-32[.]ngrok[.]io||
```

#### What Windows executable was used to download the suspicious binary?
The command line showed PowerShell pulling the payload.

### Answer
```text
||C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe||
```

## 3. Elevated Execution

#### What command was executed to configure the suspicious binary to run with elevated privileges?
The next useful field was `CommandLine`. The attacker used `schtasks.exe` to create an event-triggered task that would run as `SYSTEM`.

### SPL
```spl
index=* ComputerName="DESKTOP-TBV8NEF" "*<suspicious-binary>*" Image="C:\\Windows\\system32\\schtasks.exe"
| table UtcTime, CommandLine
| sort UtcTime
```

### Answer
```text
||"C:\Windows\system32\schtasks.exe" /Create /TN OUTSTANDING_GUTTER.exe /TR C:\Windows\Temp\COUTSTANDING_GUTTER.exe /SC ONEVENT /EC Application /MO *[System/EventID=777] /RU SYSTEM /f||
```

#### What permissions will the suspicious binary run as? What was the command to run the binary with elevated privileges?
To confirm the execution context, I searched for the suspicious task name and checked the user tied to the run event. Splunk showed the task running as `SYSTEM`.

### Answer
```text
||NT AUTHORITY\SYSTEM;"C:\Windows\system32\schtasks.exe" /Run /TN OUTSTANDING_GUTTER.exe||
```

## 4. Remote Callback

#### The suspicious binary connected to a remote server. What address did it connect to?
With the binary identified, I pivoted into network-related events and found the callback address. It used another ngrok URL.

### SPL
```spl
index=* EventCode=22 ComputerName="DESKTOP-TBV8NEF" Image="*<suspicious-binary>*"
| table UtcTime, Image, SourcePort, SourceIp, DestinationPort, DestinationIp, QueryName, EventCode
| sort UtcTime
```

### Answer
```text
||http://9030-181-215-214-32[.]ngrok[.]io||
```

## 5. PowerShell Script

#### A PowerShell script was downloaded to the same location as the suspicious binary. What was the name of the file?
Searching for PowerShell script activity and `.ps1` file writes revealed the script dropped alongside the binary.

### SPL
```spl
index=* ComputerName="DESKTOP-TBV8NEF" EventCode=11 TargetFilename="C:\\Windows\\Temp\\*.ps1"
| table UtcTime, TargetFilename
| sort UtcTime
```

### Answer
```text
||script.ps1||
```

#### The malicious script was flagged as malicious. What do you think was the actual name of the malicious script?
I pivoted the script hash into VirusTotal. The public detection name made the ransomware family obvious.

### SPL
```spl
index=* ComputerName="DESKTOP-TBV8NEF" "*<script-name>*"
| table UtcTime, TargetFilename, Hashes, EventCode
| sort UtcTime
```

### Answer
```text
||BlackSun.ps1||
```

## 6. Ransomware IOCs

#### A ransomware note was saved to disk, which can serve as an IOC. What is the full path to which the ransom note was saved?
The ransomware note was written under Keegan's downloads directory.

### SPL
```spl
index=* ComputerName="DESKTOP-TBV8NEF" EventCode=11 TargetFilename="*.txt"
| table UtcTime, TargetFilename
| sort UtcTime
```

### Answer
```text
||C:\Users\keegan\Downloads\vasg6b0wmw029hd\BlackSun_README.txt||
```

#### The script saved an image file to disk to replace the user's desktop wallpaper, which can also serve as an IOC. What is the full path of the image?
The script also dropped a wallpaper image under the public pictures directory.

### SPL
```spl
index=* ComputerName="DESKTOP-TBV8NEF" EventCode=11 TargetFilename IN ("*.jpg", "*.jpeg", "*.png")
| table UtcTime, TargetFilename
| sort UtcTime
```

### Answer
```text
||C:\Users\Public\Pictures\blacksun.jpg||
```

## 7. Summary
This was a ransomware attempt. Splunk showed PowerShell downloading a suspicious binary from an ngrok URL, then `schtasks.exe` configured it to run as `SYSTEM`. The binary later reached out to a second ngrok endpoint.

The follow-on PowerShell script tied the activity to BlackSun ransomware. The ransom note and wallpaper path gave clean IOCs for detection and response.
