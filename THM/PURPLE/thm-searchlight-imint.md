---
title: Searchlight - IMINT
summary: TryHackMe OSINT room covering imagery intelligence, geolocation, Google dorking, reverse image search, street view checks, and video frame analysis.
date: 2026-07-02
tags: [TryHackMe, OSINT, IMINT, GEOINT, Google Dorking, Reverse Image Search, Purple Team]
difficulty: easy
os: N/A
url: https://tryhackme.com/room/searchlightosint
---

# CTF Room: Searchlight - IMINT
- [Link to room](https://tryhackme.com/room/searchlightosint)
- **Difficulty:** Easy
- **Category:** OSINT, IMINT, GEOINT, Purple Team
- **OS:** N/A

## 1. Brief
Searchlight - IMINT is a TryHackMe OSINT room focused on imagery intelligence and geospatial intelligence. The room starts with simple visual clues, then moves into Google dorking, reverse image search, map checks, and video frame analysis.

The room uses `sl` plus braces around each answer.

For the writeup, I have spoiler-wrapped the submitted answers.

## 2. Task 1 - Flag Format

#### Did you understand the flag format?
The room gives the first answer directly so you understand how submissions work.

### Answer
```text
||sl{ready}||
```

## 3. Task 2 - First Geolocation

#### What is the name of the street where this image was taken?
This one was a good reminder to start with the image before touching tools. The street clue was visible in the scene, and the location matched Carnaby Street.

### Answer
```text
||sl{carnaby street}||
```

## 4. Task 3 - Tube Station
The next image moved from visual inspection into basic Google work. The useful clue was the Underground setting and the `Circus` wording around the station entrance.

I searched around:

```text
Circus Street London Underground
```

That led to Piccadilly Circus, and Street View confirmed the same entrance and surrounding buildings.

#### Which city is the tube station located in?
### Answer
```text
||sl{london}||
```

#### Which tube station do these stairs lead to?
### Answer
```text
||sl{piccadilly circus}||
```

#### Which year did this station open?
Once the station was identified, the opening year was quick to confirm from public station references.

### Answer
```text
||sl{1906}||
```

#### How many platforms are there in this station?
### Answer
```text
||sl{4}||
```

## 5. Task 4 - Airport

#### Which building is this photo taken in?
The key visible phrase was `YVR CONNECTS`. Searching that pointed to Vancouver International Airport.

### Answer
```text
||sl{vancouver international airport}||
```

#### Which country is this building located in?
### Answer
```text
||sl{canada}||
```

#### Which city is this building located in?
The airport is in Richmond, British Columbia.

### Answer
```text
||sl{richmond}||
```

## 6. Task 5 - Coffee And A Light Lunch
This task used two images and a Scotland clue. The first trap was the nearby Edinburgh Woollen Mill, because that is a chain with a lot of possible locations.

I narrowed the search to Scottish branches, then checked likely town/street layouts against the image. Blairgowrie matched the corner layout, the street sign, and the nearby coffee shop. The shop details then gave the contact information and owners.

#### Which city is this coffee shop located in?
### Answer
```text
||sl{blairgowrie}||
```

#### Which street is this coffee shop located in?
### Answer
```text
||sl{allan street}||
```

#### What is their phone number?
### Answer
```text
||sl{+447878 839128}||
```

#### What is their email address?
### Answer
```text
||sl{theweecoffeeshop@aol.com}||
```

#### What is the surname of the owners?
Searching for the shop name with `Blairgowrie owners` returned the surname.

### Answer
```text
||sl{cochrane}||
```

## 7. Task 6 - Reverse Image Search
This section introduced reverse image search. I cropped around the more distinctive interior detail, especially the ceiling, and used image search to find matches for the restaurant.

#### Which restaurant was this picture taken at?
The room wanted the shorter common name rather than the longer formal name.

### Answer
```text
||sl{katz's deli}||
```

#### What is the name of the Bon Appetit editor that worked 24 hours at this restaurant?
Once the restaurant was identified, searching for the Bon Appetit 24-hour article gave the editor's name.

### Answer
```text
||sl{andrew knowlton}||
```

## 8. Task 7 - Locate This Sculpture
The sculpture was unusual enough that reverse image search and some manual result checking worked well. Google image search returned a lot of room walkthrough noise, so I checked the source sites and image credits rather than trusting the first results.

#### What is the name of this statue?
### Answer
```text
||sl{rudolph the chrome nosed reindeer}||
```

#### Who took this image?
The source page credited the image to Kjersti Stensrud.

### Answer
```text
||sl{kjersti stensrud}||
```

## 9. Task 8 - Justice Statue
The statue clearly showed Lady Justice: scales, blindfold, and the usual justice symbolism.

For the location, Google image search was noisy. Yandex gave better visual matches, and one result led to the Alexandria, Virginia location. With the location identified, I opened the area in Maps and checked the building opposite the statue.

#### What is the name of the character that the statue depicts?
### Answer
```text
||sl{lady justice}||
```

#### Where is this statue located?
### Answer
```text
||sl{alexandria, virginia}||
```

#### What is the name of the building opposite from this statue?
### Answer
```text
||sl{the westin alexandria old town}||
```

## 10. Task 9 - Hotel Video
The final task used a video rather than a still image. I treated it as a series of frames and looked for skyline and hotel-view clues. FFmpeg is useful for pulling stills from a clip when the view changes quickly.

### Useful command
```bash
ffmpeg -i input.mp4 -vf fps=1 frames/out-%04d.jpg
```

The view matched Singapore around Clarke Quay, and the hotel was Novotel Singapore Clarke Quay.

#### What is the name of the hotel that my friend stayed in a few years ago?
### Answer
```text
||sl{novotel singapore clarke quay}||
```

## 11. Summary
This room is a solid IMINT basics path. The early tasks reward plain visual inspection, then the later tasks force better OSINT habits: search exact visible phrases, verify with Street View, crop reverse-image searches around distinctive details, try another search engine when results are noisy, and extract frames from video when a single paused frame is not enough.
