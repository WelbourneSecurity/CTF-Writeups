---
title: TShark Challenge II - Directory
summary: TryHackMe network forensics room using TShark to investigate directory-index browsing, identify a suspicious domain, export HTTP objects, and validate a downloaded executable in VirusTotal.
date: 2026-07-02
tags: [TryHackMe, TShark, Wireshark, PCAP, HTTP, DNS, VirusTotal, Malware Analysis]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/tsharkchallengestwo
---

# CTF Room: TShark Challenge II - Directory
- [Link to room](https://tryhackme.com/room/tsharkchallengestwo)
- **Difficulty:** Easy
- **Category:** Network Forensics, TShark, HTTP Object Extraction
- **OS:** Linux

## 1. Brief
TShark Challenge II: Directory is a TryHackMe network forensics room. The alert says a user found a poor file index, followed their curiosity, and triggered a security issue.

The capture file was located at:

```text
~/Desktop/exercise-files/directory-curiosity.pcap
```

The goal was to confirm the alert as a true positive by identifying the suspicious domain, reviewing HTTP traffic, extracting downloaded objects, and checking the malicious file in VirusTotal.

> **Warning:** The room notes that the samples are real examples. I kept analysis inside the provided VM and treated the domains, IPs, and files as potentially unsafe.

## 2. Task 1 - Introduction

#### Read the task above and start the attached VM.
No answer was required here. I started the VM and moved into the exercise files directory.

### Command
```bash
cd ~/Desktop/exercise-files
```

## 3. DNS Investigation

#### What is the name of the malicious/suspicious domain?
I started by extracting DNS query names from the capture.

### Command
```bash
tshark -r directory-curiosity.pcap -Y "dns.qry.name" -T fields \
  -e frame.number -e ip.src -e dns.qry.name | sort -u
```

After checking the domains in VirusTotal, `jx2-bavuong.com` was the suspicious one.

### Answer
```text
jx2-bavuong[.]com
```

#### What is the total number of HTTP requests sent to the malicious domain?
I filtered HTTP requests for the malicious host and counted them.

### Command
```bash
tshark -r directory-curiosity.pcap -Y 'http.request && http.host == "jx2-bavuong.com"' \
  -T fields -e frame.number -e http.request.method -e http.host -e http.request.uri | wc -l
```

### Answer
```text
14
```

#### What is the IP address associated with the malicious domain?
To map the domain to an IP, I reviewed DNS responses containing A records.

### Command
```bash
tshark -r directory-curiosity.pcap -Y "dns.flags.response == 1 && dns.a" -T fields \
  -e frame.number -e dns.qry.name -e dns.a | sort -u
```

### Answer
```text
141[.]164[.]41[.]174
```

#### What is the server info of the suspicious domain?
The HTTP server header gave the exposed server stack.

### Command
```bash
tshark -r directory-curiosity.pcap -Y 'http.host == "jx2-bavuong.com" && http.server' \
  -T fields -e http.server | sort -u
```

### Answer
```text
Apache/2.2.11 (Win32) DAV/2 mod_ssl/2.2.11 OpenSSL/0.9.8i PHP/5.2.9
```

## 4. First TCP Stream

#### What is the number of listed files?
The prompt pointed at the first TCP stream, so I followed stream `0` in ASCII.

### Command
```bash
tshark -r directory-curiosity.pcap -q -z follow,tcp,ascii,0
```

The directory listing contained three files.

### Answer
```text
3
```

#### What is the filename of the first file?
The first listed file in the directory index was `123.php`.

### Answer
```text
123[.]php
```

## 5. HTTP Object Export

#### What is the name of the downloaded executable file?
I exported the HTTP objects from the capture and reviewed the extracted files.

### Command
```bash
mkdir -p exported-http
tshark -r directory-curiosity.pcap --export-objects http,exported-http -q
ls -la exported-http
```

The downloaded executable was `vlauto.exe`.

### Answer
```text
vlauto[.]exe
```

#### What is the SHA256 value of the malicious file?
I hashed the exported executable.

### Command
```bash
sha256sum exported-http/vlauto.exe
```

### Answer
```text
||b4851333efaf399889456f78eac0fd532e9d8791b23a86a19402c1164aed20de||
```

## 6. VirusTotal Review

#### What is the PEiD packer value?
I searched the SHA256 value in VirusTotal and checked the file details.

### Answer
```text
.NET executable
```

#### What does the Lastline Sandbox flag this as?
The Lastline Sandbox result classified the file as a malware trojan.

### Answer
```text
MALWARE TROJAN
```

## 7. Summary
The DNS and HTTP traffic showed a user browsing a suspicious open directory on `jx2-bavuong.com`.

TShark confirmed 14 HTTP requests to the suspicious host, mapped the domain to `141[.]164[.]41[.]174`, and exposed the server header. Following the first TCP stream showed a three-file directory listing, and exporting HTTP objects recovered `vlauto.exe`. Its SHA256 matched VirusTotal detections for a `.NET executable` flagged as `MALWARE TROJAN`.
