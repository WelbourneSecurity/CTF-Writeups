---
title: Sakura Room
summary: TryHackMe OSINT room using source inspection, username pivoting, GitHub history, PGP metadata, blockchain lookup, social media, WiGLE, and travel geolocation.
date: 2026-07-02
tags: [TryHackMe, OSINT, GitHub, PGP, Cryptocurrency, WiGLE, GEOINT, Purple Team]
difficulty: easy
os: N/A
url: https://tryhackme.com/room/sakura
---

# CTF Room: Sakura Room
- [Link to room](https://tryhackme.com/room/sakura)
- **Difficulty:** Easy
- **Category:** OSINT, GEOINT, SOCMINT, Purple Team
- **OS:** N/A

## 1. Brief
Sakura is a TryHackMe OSINT room from the OSINT Dojo. The room follows one target across public artefacts: an image, a username, GitHub, PGP data, cryptocurrency traces, social media, WiFi data, and travel posts.

The big lesson here is pivoting. One small clue gives you the next account, the next account gives you metadata, and that metadata opens up another source.

> **Note:** I have spoiler-wrapped the submitted answers and sensitive artefacts.

## 2. Task 1 - Introduction
The first task is an introduction to the room and does not require investigation.

## 3. Task 2 - Tip-Off

#### What username does the attacker go by?
We start with an image hosted through GitHub. My first instinct with images is usually metadata or stego, but this was an SVG, so the better move was to inspect the source.

Opening the page source showed a local path containing `/home/...`, which exposed the username.

### Finding
```text
/home/||SakuraSnowAngelAiko||/...
```

### Answer
```text
||SakuraSnowAngelAiko||
```

## 4. Task 3 - Reconnaissance
With the username in hand, I pivoted into Google and GitHub. Searching the handle led to a GitHub profile, and the profile had a repository called `PGP`.

That is useful because PGP public keys often contain an identity string with a name and email address.

### Useful check
```bash
gpg --show-keys publickey.asc
```

#### What is the attacker's full email address?
The public key exposed the ProtonMail address.

### Answer
```text
||SakuraSnowAngel83@protonmail.com||
```

#### What is the attacker's full real name?
The same username also led to a LinkedIn profile. That gave the full name.

### Answer
```text
||Aiko Abe||
```

## 5. Task 4 - Unveil
The room then hints that the target has been editing or deleting information from GitHub. That means commit history is the place to go.

On the GitHub profile, the visible repositories did not tell the whole story. Checking the full repositories list exposed an `ETH` repo. From there, the commit history revealed a wallet address.

### Useful GitHub checks
```text
Repositories tab
Commit history
Deleted or edited file contents
```

#### What cryptocurrency does the attacker own a cryptocurrency wallet for?
The repo name and the transaction history pointed to the same cryptocurrency named in the answer.

### Answer
```text
||Ethereum||
```

#### What is the attacker's cryptocurrency wallet address?
The wallet address was recovered from the GitHub history.

### Answer
```text
||0xa102397dbeeBeFD8cD2F73A89122fCdB53abB6ef||
```

#### What mining pool did the attacker receive payments from on January 23, 2021 UTC?
I searched the wallet address in a blockchain explorer and checked the transactions around January 23, 2021. The sender label showed the mining pool.

### Answer
```text
||Ethermine||
```

## 6. Task 5 - Taunt
Next stop was social media. Searching the target's identity and handle led to the current Twitter/X handle.

The account pointed toward a paste containing WiFi SSIDs and passwords. The paste itself gave SSIDs, but the room asked for the BSSID of the home WiFi. For that, WiGLE was the right tool.

#### What is the attacker's current Twitter handle?
### Answer
```text
||SakuraLoverAiko||
```

#### What is the URL for the location where the attacker saved their WiFi SSIDs and passwords?
The paste was hosted as an onion URL.

### Answer
```text
||depasteon6cqgrykzrgya52xglohg5ovyuyhte3ll7hzix7h5ldfqsyd.onion/show.php?md5=0a5c6e136a98a60b8a21643ce8c15a74||
```

#### What is the BSSID for the attacker's home WiFi?
The paste listed WiFi names. I searched the home SSID in WiGLE and used the matching network ID as the BSSID.

### Answer
```text
||84:AF:EC:34:FC:F8||
```

## 7. Task 6 - Homebound
The last task uses travel posts and geolocation. At this point, the target's social accounts give enough context to track the journey home.

#### What airport is closest to the location the attacker shared a photo from prior to getting on their flight?
The pre-flight photo showed Washington, D.C. landmarks. The nearest airport to that location was Ronald Reagan Washington National Airport.

### Answer
```text
||DCA||
```

#### What airport did the attacker have their last layover in?
The lounge photo matched the Japan Airlines Sakura Lounge at Haneda.

### Answer
```text
||HND||
```

#### What lake can be seen in the map shared by the attacker as they were on their final flight home?
The in-flight map aligned with Japan, and the visible lake matched the answer below.

### Answer
```text
||Lake Inawashiro||
```

#### What city does the attacker likely consider home?
The WiFi paste had a city WiFi entry:

```text
||HIROSAKI_Free_Wi-Fi||
```

That tied the home city back to the answer below.

### Answer
```text
||Hirosaki||
```

## 8. Summary
Sakura is a good OSINT pivoting room. The investigation starts with an SVG source leak, moves into GitHub and PGP metadata, then follows public traces through blockchain records, social media, WiGLE, and travel geolocation.

The useful habit is to keep every artefact. Usernames, commit history, PGP identities, SSIDs, and airport photos all become pivots once you connect them to the same person.
