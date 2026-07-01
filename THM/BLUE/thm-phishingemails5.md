---
title: Phishing Emails 5
summary: TryHackMe phishing analysis room focused on investigating an email sample through headers, sender artifacts, SPF and DMARC checks, attachment hashing, and VirusTotal.
date: 2026-07-01
tags: [TryHackMe, Phishing, Email Analysis, Headers, SPF, DMARC, VirusTotal]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/phishingemails5fgjlzxc
---

# CTF Room: Phishing Emails 5
- [Link to room](https://tryhackme.com/room/phishingemails5fgjlzxc)
- **Difficulty:** Easy
- **Category:** Phishing, Email Analysis, Headers, Threat Intelligence
- **OS:** Linux

## 1. Brief
A sales executive at Greenholt PLC reported a suspicious email from a known customer. The message had several phishing indicators: a generic greeting, an unexpected money transfer request, and an unsolicited attachment.

The goal was to inspect the email, extract artifacts, validate the sender infrastructure, and assess the attachment.

## 2. Lab Setup
After starting the TryHackMe lab machine, I opened the provided `challenge.eml` file in Thunderbird.

For header-level checks, I also opened the `.eml` file in a text editor so I could inspect the raw message source.

## 3. Email Triage

#### What is the Transfer Reference Number listed in the email's Subject line?
The transfer reference number was visible in the Thunderbird subject line as the `TRN`.

### Answer
```text
09674321
```

#### What is the display name of the sender?
The display name was visible in Thunderbird.

### Answer
```text
Mr. James Jackson
```

#### What is the sender's email address?
I opened the `.eml` file in a text editor and inspected the real sender address in the message source.

### Answer
```text
info@mutawamarine.com
```

#### What email address will receive a reply to this email?
The reply-to address did not match the real sender domain, which was one of the key signs that the email should not be trusted.

### Answer
```text
info.mutawamarine@mail.com
```

## 4. Header Analysis

#### What is the originating IP address of this email?
In the message source, I reviewed the `Received` headers and found the originating IP associated with the `mutawamarine.com` domain.

### Answer
```text
192.119.71.157
```

#### Who is the owner of the originating IP?
I searched the IP address in a WHOIS lookup. The owner returned for the IP was HostPapa.

### Answer
```text
HostPapa
```

## 5. Domain Authentication

#### What is the full SPF record for this domain?
I checked the SPF record for the return-path domain with MXToolbox.

### SPF Record
```text
v=spf1 include:spf.protection.outlook.com -all
```

### Answer
```text
v=spf1 include:spf.protection.outlook.com -all
```

#### What is the complete DMARC record for this domain?
Using the same lookup workflow, I checked the DMARC policy for the return-path domain.

### DMARC Record
```text
v=DMARC1; p=quarantine; fo=1
```

### Answer
```text
v=DMARC1; p=quarantine; fo=1
```

## 6. Attachment Analysis

#### What is the file name of the attachment found in the email?
The attachment name was visible in Thunderbird.

### Answer
```text
SWT_#09674321____PDF__.CAB
```

#### Using the sha256sum command, what is the SHA256 hash of the file?
I downloaded the attachment into the lab VM and generated its SHA256 hash.

### Command
```bash
sha256sum SWT_#09674321____PDF__.CAB
```

### Output
```text
||2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f||  SWT_#09674321____PDF__.CAB
```

### Answer
```text
||2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f||
```

#### What is the attachment's file size in KB?
I searched the SHA256 hash in VirusTotal and checked the file details.

### VirusTotal
```text
https://www.virustotal.com/gui/home/search
```

### Answer
```text
400.26 KB
```

#### What is the actual file type of the attachment?
VirusTotal identified the real file type in the details tab.

### Answer
```text
RAR
```

## 7. Summary
This email showed multiple phishing indicators: a suspicious reply-to mismatch, an unexpected transfer request, and an attachment whose extension did not clearly reflect its actual archive type.

Inspecting the raw headers, validating SPF and DMARC records, hashing the attachment, and checking the hash in VirusTotal gave enough evidence to treat the message as malicious.
