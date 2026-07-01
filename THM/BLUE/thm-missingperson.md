---
title: Missing Person
summary: TryHackMe OSINT room using image search, event research, EXIF metadata, social media, and public business details to trace a missing person's travel trail.
date: 2026-07-01
tags: [TryHackMe, OSINT, GEOINT, EXIF, SOCMINT, Google Lens]
difficulty: easy
os: N/A
url: https://tryhackme.com/room/missingperson
---

# CTF Room: Missing Person
- [Link to room](https://tryhackme.com/room/missingperson)
- **Difficulty:** Easy
- **Category:** OSINT, GEOINT, Image Analysis
- **OS:** N/A

## 1. Brief
Missing Person is a TryHackMe OSINT room. The scenario provides holiday photos from 2025 and asks us to reconstruct where the missing person went, what event they attended, and who they contacted before disappearing.

The task files included:

```text
food.jpg
MotoGP.jpg
```

The most useful workflow was image search, metadata inspection, event research, and social media pivoting.

> **Tip:** For this room, Google Lens was the fastest way to turn image context into search terms and locations.

## 2. Circuit Identification

#### What is the commercial name of this circuit?
The MotoGP image pointed to the Mandalika circuit in Lombok, Indonesia. Image search matched it to the circuit's commercial name.

### Answer
```text
Pertamina Mandalika International Street Circuit
```

#### When did the event take place?
The photo context and 2025 MotoGP calendar narrowed the event to the Mandalika race weekend.

### Answer
```text
03-05/10/2025
```

## 3. Food Photo Analysis

#### He told me he ate delicious Mexican food. What is the name of the restaurant?
Running the food photo through Google Lens matched the restaurant to Cantina Mexicana.

### Answer
```text
Cantina Mexicana
```

#### At what time was this photo taken?
I checked the image metadata with Jimpl.

### Tool
```text
https://jimpl.com/
```

### Answer
```text
19:55:30
```

## 4. MotoGP After Party

#### What is the full address of the bar's location?
The final message mentioned a MotoGP after party. Searching for `After Party MotoGP Mandalika 2025` led to an event listing for `After Race Party Mandalika GP Vol.3` at Mandalika Beach Club.

That looked promising, but the DJ trail did not line up with the answer format. The useful pivot was a separate MotoGP after-party at Surfers Bar Kuta Lombok.

### Answer
```text
Jl. Raya Kuta, Kuta, Kec. Pujut, Kabupaten Lombok Tengah, Nusa Tenggara Bar
```

#### What is the DJ's stage name?
The Mandalika Beach Club event was a false lead. It featured a headliner, DJ Panda, and other DJs in the event listing, but none matched the expected answer format.

The breakthrough came from looking at the Surfers Bar Kuta Lombok MotoGP after-party and checking their social media reels. The local DJ who played that night was BONG LELEH.

### Answer
```text
BONG LELEH
```

## 5. Social Media Pivot

#### After digging into the DJ's other online accounts, what cave does he take tourists to?
Pivoting from the DJ's public accounts showed that he was associated with local tours. The cave clue narrowed the answer to Gua Sumur.

### Answer
```text
Gua Sumur
```

#### What number did the DJ list for his tour business?
To finish the trail, I used search pivots from the DJ's stage name to find his public social accounts. His Facebook profile listed a phone number for the tour business.

### Answer
```text
||085333137345||
```

## 6. Summary
The investigation started with image recognition for the MotoGP circuit and restaurant, then used metadata to recover the photo time.

The after-party clue required avoiding an early false lead and pivoting to a separate local MotoGP party at Surfers Bar Kuta Lombok. From there, social media activity connected the DJ to local cave tours and the listed tour business number.
