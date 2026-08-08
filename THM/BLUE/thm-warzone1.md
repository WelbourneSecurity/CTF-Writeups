---
title: Warzone 1
summary: TryHackMe SOC triage room investigating an IDS alert with Brim, Wireshark, NetworkMiner, and VirusTotal to confirm malware command and control activity.
date: 2026-07-02
tags: [TryHackMe, SOC, PCAP, Brim, Wireshark, NetworkMiner, VirusTotal, Malware]
difficulty: medium
os: Windows
url: https://tryhackme.com/room/warzoneone
---

# CTF Room: Warzone 1
- [Link to room](https://tryhackme.com/room/warzoneone)
- **Difficulty:** Medium
- **Category:** SOC, PCAP Analysis, Malware Traffic Analysis
- **OS:** Windows

## 1. Brief
Warzone 1 is a TryHackMe SOC triage room. The scenario starts with an IDS/IPS alert for potentially bad traffic and malware command and control activity.

The goal was to inspect the PCAP, confirm whether the alert was a true positive, and pull out the useful artefacts.

Tools used:

- Brim
- Wireshark
- NetworkMiner
- VirusTotal
- CyberChef for defanging

## 2. Alert Triage

#### What was the alert signature for Malware Command and Control Activity Detected?
I loaded the PCAP into Brim and searched for the malware command and control alert. Keeping it simple worked here: search the alert wording and check the signature field.

### Answer
```text
||ET MALWARE MirrorBlast CnC Activity M3||
```

#### What is the source IP address?
The alert showed the internal source host.

### Answer
```text
||172[.]16[.]1[.]102||
```

#### What IP address was the destination IP in the alert?
The same alert showed the external destination.

### Answer
```text
||169[.]239[.]128[.]11||
```

## 3. VirusTotal Enrichment

#### Under Community, what threat group is attributed to this IP address?
I searched the flagged destination IP in VirusTotal and checked the Community tab. The IP was associated with a known threat group.

### Answer
```text
||TA505||
```

#### What is the malware family?
The alert and VT context both pointed to the same malware family.

### Answer
```text
||MirrorBlast||
```

#### What was the majority file type listed under Communicating Files?
Under VirusTotal's communicating files view, the majority type lined up with the installer payloads later recovered from the PCAP.

### Answer
```text
||Windows Installer||
```

## 4. Web Traffic Review

#### Inspect the web traffic for the flagged IP address; what is the user-agent in the traffic?
In Wireshark, I filtered on the flagged IP and searched packet details for `User-Agent`.

### Display filter
```text
ip.addr == <flagged-ip>
```

The HTTP traffic used a distinctive user-agent.

### Answer
```text
||REBOL View 2.7.8.3.1||
```

## 5. Retracing The Attack

#### There were multiple IP addresses associated with this attack. What were two other IP addresses?
Back in Brim, I filtered HTTP traffic and looked for unique source, destination, and user-agent combinations.

### Brim query
```text
_path == "http" | cut id.orig_h, id.resp_h, user_agent | uniq
```

Two more external IPs stood out because they shared the same suspicious installer activity.

### Answer
```text
||185[.]10[.]68[.]235,192[.]36[.]27[.]92||
```

#### What were the file names of the downloaded files?
I loaded the PCAP into NetworkMiner and checked the `Files` tab. Filtering by the two IPs from the previous question showed the MSI downloads.

### Answer
```text
||filter.msi,10opd3r_load.msi||
```

## 6. Payload Paths

#### Inspect the traffic for the first downloaded file. What is the full file path of the directory and the name of the two files?
For the first MSI download, I went back into Wireshark, filtered by the relevant IP, and followed the HTTP stream. The stream exposed plaintext strings containing the file paths that would be saved on disk.

### Answer
```text
||C:\ProgramData\001\arab.bin,C:\ProgramData\001\arab.exe||
```

#### Inspect the traffic for the second downloaded file. What is the full file path of the directory and the name of the two files?
I repeated the same process for the second MSI download: filter by IP, follow the HTTP stream, and inspect the payload strings.

### Answer
```text
||C:\ProgramData\Local\Google\rebol-view-278-3-1.exe,C:\ProgramData\Local\Google\exemple.rb||
```

## 7. Summary
This alert was a true positive. Brim gave the initial IDS signature and traffic pivots, VirusTotal tied the infrastructure to the threat group and malware family, Wireshark exposed the user-agent and payload paths, and NetworkMiner made the downloaded MSI files easy to confirm.

The chain was clean: internal host to flagged C2, related MSI downloads from two more IPs, and file paths showing what those installers tried to place under `C:\ProgramData`.
