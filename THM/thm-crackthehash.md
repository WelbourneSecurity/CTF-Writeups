---
title: Crack the Hash
summary: TryHackMe hash cracking room covering common hashes, CrackStation lookups, and Hashcat modes for bcrypt, SHA-512 crypt, and salted SHA1.
date: 2026-06-29
difficulty: easy
os: N/A
tags: [TryHackMe, Cryptography, Hash Cracking, CrackStation, Hashcat, John the Ripper]
url: https://tryhackme.com/room/crackthehash
badge: https://tryhackme.com/dashboard?badge=welbournesecurity:hash-cracker
---

# CTF Room: Crack the Hash
- [Link to room](https://tryhackme.com/room/crackthehash)
- [Badge earned: Hash Cracker](https://tryhackme.com/dashboard?badge=welbournesecurity:hash-cracker)
- **Difficulty:** Easy
- **Category:** Cryptography, Hash Cracking

## 1. Brief
Crack the Hash is a TryHackMe room focused on identifying common hash formats and recovering plaintext passwords.

For most of these hashes, I used [CrackStation](https://crackstation.net/) first because it is quick for common unsalted hashes. When the hashes were slower, salted, or not available through online lookup, I switched to John the Ripper and Hashcat with `rockyou.txt`.

## 2. Level 1
The first level contains a set of easier hashes that can mostly be cracked with online lookup tools.

#### 48bb6e862e54f2a795ffc4e541caed4d
This hash was cracked using CrackStation.

### Answer
```bash
easy
```

#### CBFDAC6008F9CAB4083784CBD1874F76618D2A97
This hash was cracked using CrackStation.

### Answer
```bash
password123
```

#### 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
This hash was cracked using CrackStation.

### Answer
```bash
letmein
```

#### $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
This is a bcrypt hash. Bcrypt is intentionally slow, so this one is better suited to a local cracking tool such as Hashcat.

### Hashcat Command
```bash
hashcat -m 3200 -a 0 bcrypt.txt /usr/share/wordlists/rockyou.txt
```

### Answer
```bash
bleh
```

#### 279412f945939ba78ce0758d3fd83daa
This hash was cracked using CrackStation.

### Answer
```bash
Eternity22
```

## 3. Level 2
Level 2 increases the difficulty. The room notes that all answers are in the classic `rockyou.txt` password list, so Hashcat is useful here.

#### F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
This hash was cracked with `rockyou.txt`.

### Answer
```bash
paule
```

#### 1DFECA0C002AE40B8619ECF94819CC1B
This hash was cracked with `rockyou.txt`.

### Answer
```bash
n63umy8lkf4i
```

#### $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.
This is a SHA-512 crypt hash. The `$6$` prefix identifies the format, and the salt is included inside the hash.

### Hash Details
```bash
Hash type: SHA-512 crypt
Hashcat mode: 1800
Salt: aReallyHardSalt
```

### Hashcat Command
```bash
hashcat -m 1800 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

### Answer
```bash
waka99
```

#### e5d8870e5bdd26602cab8dbe07a942c8669e56d6
This is a salted SHA1 hash using the format `sha1($pass.$salt)`.

### Hash Details
```bash
Hash type: salted SHA1
Hashcat mode: 110
Salt: tryhackme
```

### Hashcat Command
```bash
echo 'e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme' > sha1salt.txt
hashcat -m 110 -a 0 sha1salt.txt /usr/share/wordlists/rockyou.txt
```

### Answer
```bash
481616481616
```

## 4. Useful Hashcat Modes
These were the main Hashcat modes needed for the room:

```bash
# SHA-512 crypt
hashcat -m 1800 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# Salted SHA1: hash:salt
hashcat -m 110 -a 0 sha1salt.txt /usr/share/wordlists/rockyou.txt

# bcrypt
hashcat -m 3200 -a 0 bcrypt.txt /usr/share/wordlists/rockyou.txt
```

## 5. Summary
This was a good beginner-friendly hash cracking room. The first level can mostly be completed with CrackStation, which is useful for quickly checking common hashes.

The second level is where local cracking becomes more useful. Hashcat makes it much easier to handle slower formats such as bcrypt, Unix-style SHA-512 crypt hashes, and salted hashes where the salt needs to be supplied in the correct format.
