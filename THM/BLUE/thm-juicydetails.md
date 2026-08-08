---
title: Juicy Details
summary: TryHackMe SOC log-analysis room investigating attacker reconnaissance, brute forcing, SQL injection, file retrieval, and shell access against a Juice Shop environment.
date: 2026-07-01
tags: [TryHackMe, SOC, Log Analysis, Web, Brute Force, SQL Injection, FTP, SSH]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/juicydetails
---

# CTF Room: Juicy Details
- [Link to room](https://tryhackme.com/room/juicydetails)
- **Difficulty:** Easy
- **Category:** SOC, Log Analysis, Web Attacks
- **OS:** Linux

## 1. Brief
Juicy Details is a TryHackMe SOC investigation room. The scenario provides server logs from a compromised Juice Shop environment and asks us to reconstruct attacker activity.

The main goals were to identify:

- The tools and techniques used by the attacker
- The vulnerable endpoints
- The data accessed or stolen
- The services and accounts used after the web attack

## 2. Task 1 - Introduction

#### Are you ready?
After downloading the task files, I confirmed readiness.

### Answer
```text
I am ready!
```

## 3. Available Logs
The downloaded archive contained three useful log files:

```text
access.log
auth.log
vsftpd.log
```

The web attack activity was mostly visible in `access.log`, file downloads were confirmed in `vsftpd.log`, and shell access was confirmed in `auth.log`.

## 4. Task 2 - Reconnaissance

#### What tools did the attacker use? (Order by the occurrence in the log)
I reviewed `access.log` and identified tool user agents in timestamp order.

### Evidence
```text
Nmap        [11/Apr/2021:09:08:35 +0000]
Hydra       [11/Apr/2021:09:16:27 +0000]
sqlmap      [11/Apr/2021:09:29:15 +0000]
Curl        [11/Apr/2021:09:32:51 +0000]
feroxbuster [11/Apr/2021:09:34:33 +0000]
```

### Answer
```text
nmap, hydra, sqlmap, curl, feroxbuster
```

#### What endpoint was vulnerable to a brute-force attack?
Hydra traffic in `access.log` targeted the login endpoint.

### Evidence
```text
"GET /rest/user/login HTTP/1.0" 500 - "-" "Mozilla/5.0 (Hydra)"
"POST /rest/user/login HTTP/1.0" 401 26 "-" "Mozilla/5.0 (Hydra)"
```

### Answer
```text
/rest/user/login
```

#### What endpoint was vulnerable to SQL injection?
The `sqlmap` traffic targeted the product search endpoint.

### Evidence
```text
::ffff:192.168.10.5 - - [11/Apr/2021:09:29:15 +0000] "GET /rest/products/search?q=6813 HTTP/1.1" 200 30 "-" "sqlmap/1.5.2#stable (http://sqlmap.org)"
```

### Answer
```text
/rest/products/search
```

#### What parameter was used for the SQL injection?
The SQL injection payloads were passed through the `q` parameter.

### Answer
```text
q
```

#### What endpoint did the attacker try to use to retrieve files?
Feroxbuster discovered the `/ftp` endpoint, and later FTP activity in `vsftpd.log` supported that the attacker used it to retrieve files.

### Answer
```text
/ftp
```

## 5. Task 3 - Stolen Data

#### What section of the website did the attacker use to scrape user email addresses?
The attacker accessed product reviews, which exposed user email addresses.

### Evidence
```text
"GET /rest/products/26/reviews HTTP/1.1" 200
```

### Answer
```text
product reviews
```

#### Was their brute-force attack successful? If so, what is the timestamp of the successful login?
The Hydra activity showed a successful login attempt around `09:16:31`.

### Evidence
```text
::ffff:192.168.10.5 - - [11/Apr/2021:09:16:31 +0000] "GET /rest/user/login HTTP/1.0" 500 - "-" "Mozilla/5.0 (Hydra)"
::ffff:192.168.10.5 - - [11/Apr/2021:09:16:31 +0000] "POST /rest/user/login HTTP/1.0" 401 26 "-" "Mozilla/5.0 (Hydra)"
```

### Answer
```text
Yay, 11/Apr/2021:09:16:31 +0000
```

#### What user information was the attacker able to retrieve from the endpoint vulnerable to SQL injection?
The attacker used a `UNION SELECT` query against the `Users` table and selected the `email` and `password` columns.

### Evidence
```text
::ffff:192.168.10.5 - - [11/Apr/2021:09:31:04 +0000] "GET /rest/products/search?q=qwert%27))%20UNION%20SELECT%20id,%20email,%20password,%20%274%27,%20%275%27,%20%276%27,%20%277%27,%20%278%27,%20%279%27%20FROM%20Users-- HTTP/1.1" 200 - "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
::ffff:192.168.10.5 - - [11/Apr/2021:09:32:51 +0000] "GET /rest/products/search?q=qwert%27))%20UNION%20SELECT%20id,%20email,%20password,%20%274%27,%20%275%27,%20%276%27,%20%277%27,%20%278%27,%20%279%27%20FROM%20Users-- HTTP/1.1" 200 3742 "-" "curl/7.74.0"
```

### Answer
```text
email, password
```

#### What files did they try to download from the vulnerable endpoint?
The FTP logs showed two successful downloads.

### Evidence
```text
Sun Apr 11 09:35:45 2021 [pid 8154] [ftp] OK DOWNLOAD: Client "::ffff:192.168.10.5", "/www-data.bak", 2602 bytes, 544.81Kbyte/sec
Sun Apr 11 09:36:08 2021 [pid 8154] [ftp] OK DOWNLOAD: Client "::ffff:192.168.10.5", "/coupons_2013.md.bak", 131 bytes, 3.01Kbyte/sec
```

### Answer
```text
coupons_2013.md.bak, www-data.bak
```

#### What service and account name were used to retrieve files from the previous question?
The `vsftpd.log` file showed that the attacker logged in to FTP using the anonymous account.

### Evidence
```text
Sun Apr 11 09:35:37 2021 [pid 8152] [ftp] OK LOGIN: Client "::ffff:192.168.10.5", anon password "?"
```

### Answer
```text
ftp, anonymous
```

#### What service and username were used to gain shell access to the server?
The `auth.log` file showed a successful login for the `www-data` account over SSH.

### Answer
```text
ssh, www-data
```

## 6. Summary
The attacker moved through a clear chain: reconnaissance with Nmap, brute forcing the login endpoint with Hydra, SQL injection against product search, direct data extraction with curl, directory discovery with Feroxbuster, FTP file retrieval, and finally SSH access as `www-data`.

The important vulnerable endpoints were `/rest/user/login`, `/rest/products/search`, and `/ftp`.
