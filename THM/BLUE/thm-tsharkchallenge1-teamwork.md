---
title: TShark Challenge I - Teamwork
summary: TryHackMe network forensics room using TShark to investigate DNS traffic, identify a suspicious phishing domain, pivot through VirusTotal, and extract an email address from a PCAP.
date: 2026-07-02
tags: [TryHackMe, TShark, Wireshark, PCAP, DNS, VirusTotal, Threat Intelligence]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/tsharkchallengesone
---

# CTF Room: TShark Challenge I - Teamwork
- [Link to room](https://tryhackme.com/room/tsharkchallengesone)
- **Difficulty:** Easy
- **Category:** Network Forensics, TShark, DNS Analysis
- **OS:** Linux

## 1. Brief
TShark Challenge I: Teamwork is a TryHackMe network forensics room. The SOC alert says a suspicious domain may pose a threat to the organisation, and the job is to analyse the provided PCAP and create useful detection artefacts.

The capture file was located at:

```text
~/Desktop/exercise-files/teamwork.pcap
```

> **Warning:** The room notes that the exercise files contain real examples. I kept the analysis inside the provided VM and did not interact with the domains outside the lab workflow.

## 2. Task 1 - Introduction

#### Read the task above and start the attached VM.
No answer was required here. I started the VM and moved into the exercise files directory.

### Command
```bash
cd ~/Desktop/exercise-files
```

## 3. Domain Investigation

#### What is the full URL of the malicious/suspicious domain address?
I started by extracting DNS query names from the PCAP.

### Command
```bash
tshark -r teamwork.pcap -Y "dns.qry.name" -T fields \
  -e frame.number -e ip.src -e dns.qry.name | sort -u
```

One domain stood out because it was clearly trying to look like PayPal while actually living under another domain.

### IOC
```text
www.paypal.com4uswebappsresetaccountrecovery.timeseaways.com
```

After checking the domain in VirusTotal, I defanged the suspicious URL for the answer.

### Answer
```text
https://www[.]paypal[.]com4uswebappsresetaccountrecovery[.]timeseaways[.]com/
```

#### When was the URL of the malicious/suspicious domain address first submitted to VirusTotal?
VirusTotal showed the first submission timestamp for the suspicious URL.

### Answer
```text
2017-04-17 22:52:53 UTC
```

#### Which known service was the domain trying to impersonate?
The fake domain was built to look like a PayPal account recovery page.

### Answer
```text
paypal
```

## 4. Malicious Domain IP

#### What is the IP address of the malicious domain?
To resolve the IP from the capture, I filtered for DNS responses containing A records.

### Command
```bash
tshark -r teamwork.pcap -Y "dns.flags.response == 1 && dns.a" -T fields \
  -e frame.number -e dns.qry.name -e dns.a | sort -u
```

The suspicious domain resolved to the following IP.

### Answer
```text
184[.]154[.]127[.]226
```

## 5. Email Address Extraction

#### What is the email address that was used?
I searched the verbose TShark output for email-address-shaped strings.

### Command
```bash
tshark -r teamwork.pcap -V | grep -Eio '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' | sort -u
```

The output contained a lot of false positives from binary-ish packet data, but one real address stood out.

### Relevant Output
```text
johnny5alive@gmail.com
```

The room asked for the address in defanged format.

### Answer
```text
||johnny5alive[at]gmail[.]com||
```

## 6. Summary
The investigation started by extracting DNS queries from `teamwork.pcap`. A suspicious PayPal-themed domain was identified and checked in VirusTotal, where it was marked malicious or suspicious.

From there, DNS response records gave the associated IP address, and a broad TShark/grep pass over verbose packet output revealed the email address used in the traffic.
