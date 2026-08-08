---
title: Warzone 2
summary: TryHackMe SOC triage room investigating IDS alerts for a network trojan, privacy-policy violations, malicious downloads, and related suspicious infrastructure.
date: 2026-07-02
tags: [TryHackMe, SOC, PCAP, Brim, Wireshark, NetworkMiner, VirusTotal, Malware]
difficulty: medium
os: Windows
url: https://tryhackme.com/room/warzonetwo
---

# CTF Room: Warzone 2
- [Link to room](https://tryhackme.com/room/warzonetwo)
- **Difficulty:** Medium
- **Category:** SOC, PCAP Analysis, IDS Triage, Malware Traffic Analysis
- **OS:** Windows

## 1. Brief
Warzone 2 is another TryHackMe SOC triage room. The alert set includes miscellaneous activity, a network trojan detection, and a possible corporate privacy violation.

The job was to inspect the PCAP, decide whether the alert was a true positive, and recover the network artefacts.

Tools used:

- Brim
- Wireshark
- NetworkMiner
- VirusTotal
- CyberChef for defanging

## 2. Alert Triage

#### What was the alert signature for A Network Trojan was Detected?
I loaded `Zone2.pcap` into Brim and started with Suricata alert grouping. This quickly showed the IP involved in the network trojan alert.

### Brim query
```text
event_type=="alert" | alerts := union(alert.category) by src_ip, dest_ip
```

I then searched the triggered IP in Brim and opened the alert record to read the signature.

### Answer
```text
||ET MALWARE Likely Evil EXE download from MSXMLHTTP non-exe extension M2||
```

#### What was the alert signature for Potential Corporate Privacy Violation?
The privacy violation alert used the same triggered IP. Opening the matching alert record gave the policy signature.

### Answer
```text
||ET POLICY PE EXE or DLL Windows file download HTTP||
```

#### What was the IP to trigger either alert?
The same IP triggered both alert categories.

### Answer
```text
||185[.]118[.]164[.]8||
```

## 3. Malicious Download

#### Provide the full URI for the malicious downloaded file.
I moved from the alert into HTTP traffic. Brim showed the request URI for the suspicious download, and CyberChef handled the defanging.

### Brim query
```text
_path=="http" | cut id.orig_h, id.resp_h, id.resp_p, method, host, uri, status_code | uniq -c | sort status_code
```

### Answer
```text
||awh93dhkylps5ulnq-be[.]com/czwih/fxla[.]php?l=gap1[.]cab||
```

#### What is the name of the payload within the cab file?
The CAB payload name was visible from the file details and could also be confirmed by pivoting the signature hash into VirusTotal.

### Answer
```text
||draw.dll||
```

#### What is the user-agent associated with this network traffic?
I opened the HTTP request in Brim and checked the `user_agent` field.

### Answer
```text
||Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; WOW64; Trident/8.0; .NET4.0C; .NET4.0E)||
```

## 4. Related Malicious Domains

#### What other domains do you see in the network traffic that are labelled as malicious by VirusTotal?
The HTTP summary query showed more domains in the capture. I checked the domains in VirusTotal, kept the ones marked malicious, then sorted them alphabetically.

### Answer
```text
||a-zcorner[.]com,knockoutlights[.]com||
```

## 5. Not Suspicious Traffic Alerts

#### There are IP addresses flagged as Not Suspicious Traffic. What are the IP addresses?
Back in Brim's Suricata alert grouping, two IPs were labelled under `Not Suspicious Traffic`. The label is a bit misleading for a triage room, so I still treated them as pivots and checked the associated DNS/HTTP traffic.

### Answer
```text
||64[.]225[.]65[.]166,142[.]93[.]211[.]176||
```

#### For the first IP address flagged as Not Suspicious Traffic, what domains did you spot in the network traffic?
For the first IP, I searched the PCAP for DNS queries and domains tied to that host.

### Brim query
```text
<first-not-suspicious-ip> | cut query
```

VirusTotal marked several associated domains as malicious. In alphabetical order:

### Answer
```text
||safebanktest[.]top,tocsicambar[.]xyz,ulcertification[.]xyz||
```

#### For the second IP marked as Not Suspicious Traffic, what domain did you spot in the network traffic?
I repeated the same pivot for the second IP.

### Brim query
```text
<second-not-suspicious-ip> | cut query
```

### Answer
```text
||2partscow[.]top||
```

## 6. Summary
This alert was a true positive. The Suricata alerts identified an evil EXE download pattern and a PE/DLL download over HTTP. Brim provided the alert and HTTP pivots, VirusTotal confirmed the wider malicious infrastructure, and the CAB payload showed the downloaded DLL.

The useful flow was alert signature first, then triggered IP, then HTTP URI, then VirusTotal enrichment, then related domains from DNS and HTTP traffic.
