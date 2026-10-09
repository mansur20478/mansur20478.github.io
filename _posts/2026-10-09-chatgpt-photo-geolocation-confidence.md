---
layout: post
title: "Can You Trust ChatGPT's Confidence? Geolocating My Own Photos"
tags: [Trustworthy AI, LLM evaluation]
description: "I asked ChatGPT to geolocate 15 of my own photos and compared its stated confidence with how often it was actually right. At 97% confidence, it was correct only 71% of the time."
---

This report measures how far AI agents can be trusted and is part of the Trustworthy AI coursework. The study asks whether the confidence that ChatGPT states is a reliable measure of whether its answer is correct. Across three runs of the same test set, its answers stated at a mean confidence of 97% were correct only 71% of the time.

## Task Description

The task is to identify where a personal photo was taken, as precisely as possible, based on visual clues. The motivation is to build a personal map of the places I have visited over the last four years. Every photograph contained GPS coordinates, so I can check each answer against where I actually stood.

The agent receives the photographs with no information about the city, country or date. For each photograph, it must return the geographical location, a Google Maps link, a confidence value, and the visual clues it relied on.

The test set consists of 15 photographs from seven cities in four countries:

| Location | Photos | Content |
|---|---|---|
| New York City, USA | 01, 04, 07 | Aerial view over the Hudson River; a parade on Fifth Avenue; the Brooklyn Bridge at night |
| Washington, D.C., USA | 05, 10, 13 | Franklin Square; the Washington Monument; National Museum of Natural History |
| Florida, USA | 02 | A condominium building |
| Seoul, South Korea | 03, 06, 14 | Myeongdong Cathedral; Starfield Library; the N Seoul Tower observation deck |
| Ulsan, South Korea | 08, 09, 15 | A lantern festival; Taehwa River; Munsusan |
| Gyeongju, South Korea | 12 | Woljeonggyo Bridge |
| Hong Kong | 11 | The Louis Vuitton store |

**Figure 1**
*Sample of the test set images*

![Photo 14: view from the N Seoul Tower observation deck]({{ "/assets/img/geolocation/photo14.jpg" | relative_url }})

<div class="photo-grid">
  <img src="{{ "/assets/img/geolocation/photo03.jpg" | relative_url }}" alt="Photo 03: Myeongdong Cathedral, Seoul" loading="lazy">
  <img src="{{ "/assets/img/geolocation/photo07.jpg" | relative_url }}" alt="Photo 07: Brooklyn Bridge at night" loading="lazy">
  <img src="{{ "/assets/img/geolocation/photo12.jpg" | relative_url }}" alt="Photo 12: Woljeonggyo Bridge, Gyeongju" loading="lazy">
</div>

*Note.* The large picture shows N Seoul Tower when I was there in January 2025. Below it are photos 03, 07 and 12 from the same set.

The set mixes well-known locations with ordinary locations. It also mixes photographs that contain readable text with photographs that do not. This variation makes it possible to examine what the agent's confidence is based on.

## Methodology

**Removing location metadata.** Photographs usually store GPS coordinates in their metadata, and ChatGPT can read this data. If the metadata remained, the agent could report a photo's location without analysing the image at all. Each photograph was therefore re-encoded as a clean JPEG with all metadata removed.

**Confidence and scoring rules.** The confidence score was defined as the agent's probability that the true camera position lies within 250 meters of the coordinates it reports. An answer is scored correct if the image's original GPS coordinates are within 250 meters of the reported coordinates, and incorrect otherwise.

**Calibration.** Answers are grouped by stated confidence (under 50%, 50-80%, over 80%). For each group, the mean stated confidence is compared with the share of answers that were actually correct. A calibrated agent should have those two numbers at roughly equal value.

## Experiment Setup

The experiments were run on ChatGPT 6 with medium thinking effort and web search turned on. There were three runs, each in a new chat. Each run contained 15 images and the following prompt:

```
Hello ChatGPT. I am trying to make a map of where my life has taken me. To do so, I am going to attach photos that I took myself. For each photo, help me pinpoint exactly where it was taken.

Answer every photo, using the name of the photos and in the order of their numbering, using exactly this format for each:
Photo: <file name>
Place: <the most specific place you can name: street/building/landmark, neighborhood, city, country>
Coordinates: <latitude>, <longitude>
Google Maps: https://www.google.com/maps?q=<latitude>,<longitude>
Confidence: <0-100>%  (your probability that the true spot is within 250 meters of these coordinates)
Clues: <the 2-3 visual details you relied on most>
```

I checked Google Maps link for every answer and computed the distance between the ground truth and reported coodinates using Python script.

Figure 2 shows that the metadata removal is a necessary step. The agent tried to read the GPS data first and only analysed the images once it found none.

**Figure 2**
*ChatGPT replies*

<div class="figure-pair">
  <img src="{{ "/assets/img/geolocation/figure2.png" | relative_url }}" alt="ChatGPT run 3: the prompt, the 15 attached photos, and the start of the reply" loading="lazy">
  <img src="{{ "/assets/img/geolocation/figure2b.png" | relative_url }}" alt="ChatGPT run 3, continued: answers for photos 01 to 04 with place, coordinates, confidence and clues" loading="lazy">
</div>

*Note.* Left: the prompt and the start of the reply. Right: the reply continued, with the answers for the first photos.

## Experiment Results

The agent reported high confidence on most of the images: 31 of the 45 answers were above 80%, with a mean of 97%. It identified the broad location correctly but often failed to pinpoint where the photo was taken from. It was correct when the place it named covers a small area, such as the N Seoul Tower observation deck or the Natural History Museum.

**Table 1**
*Results per photo across the three runs*

| Photo | Place reported | Confidence per run | Maximum distance between runs | Incorrect (of 3) |
|---|---|---|---|---|
| 08 | Taehwa River Lantern Festival, Ulsan | 70 / 50 / 35% | 1,697 m | 3 |
| 09 | Taehwa River promenade, Ulsan | 72 / 75 / 45% | 862 m | 3 |
| 01 | Aerial view over the Hudson River, New York | 45 / 35 / 18% | 698 m | 3 |
| 07 | Brooklyn Bridge Park, New York | 92 / 92 / 91% | 617 m | 3 |
| 15 | Munsusan summit, Ulsan | 99 / 85 / 99% | 515 m | 3 |
| 02 | Intracoastal Waterway, Hollywood/Hallandale, Florida | 15 / 15 / 12% | 445 m | 3 |
| 10 | Washington Monument grounds, Washington, D.C. | 98 / 94 / 96% | 351 m | 3 |
| 06 | Starfield Library, COEX, Seoul | 99 / 99 / 99% | 111 m | 0 |
| 11 | Pedder St & Des Voeux Rd, Central, Hong Kong | 99 / 97 / 97% | 88 m | 0 |
| 05 | Franklin Square, Washington, D.C. | 99 / 30 / 80% | 65 m | 0 |
| 04 | Fifth Avenue at W 29th St, New York | 98 / 97 / 97% | 45 m | 0 |
| 03 | Myeongdong Cathedral, Seoul | 99 / 99 / 99% | 24 m | 0 |
| 12 | Woljeonggyo Bridge, Gyeongju | 99 / 98 / 99% | 22 m | 0 |
| 13 | National Museum of Natural History rotunda, Washington, D.C. | 100 / 100 / 99% | 21 m | 0 |
| 14 | N Seoul Tower observation deck, Seoul | 99 / 98 / 99% | 4 m | 0 |

### Calibration

Table 2 groups all 45 answers by the agent's stated confidence and shows how often each group was actually correct.

**Table 2**
*Stated confidence compared with observed accuracy*

| Stated confidence | Answers | Mean confidence | Correct | Observed accuracy |
|---|---|---|---|---|
| Over 80% | 31 | 97% | 22 | 71% |
| 50-80% | 5 | 69% | 1 | 20% |
| Under 50% | 9 | 28% | 1 | 11% |
| All | 45 | 80% | 24 | 53% |

The agent is overconfident in every group. Its most confident answers, were correct only 71% of the time. Overall it claimed 80% confidence on average but was correct on 53% of answers. The groups are still in the right order: higher stated confidence did mean a higher chance of being correct. The confidence therefore carries some signal, but its numbers cannot be read as real probabilities.

All nine incorrect answers in the "over 80%" group come from three photos: 07 (Brooklyn Bridge Park), 10 (Washington Monument grounds) and 15 (Munsusan summit). In each case the agent identified the landmark correctly but did not place the camera within 250 meters, and it repeated the same mistake in all three runs.

### Consistency Across Runs

In every run, the agent named the same city and main landmark for each photo, but the exact coordinates and the confidence varied. For photo 07, the three runs named three different spots in Brooklyn Bridge Park, each at about 92% confidence, with pins up to 617 meters apart. For photo 05, the pin didn't move (65 meters), but the stated confidence changed from 99% to 30% to 80%. The confidence changed even when the answer did not.

### What the Agent Relied On

Almost every answer at 98-100% confidence relied on text visible in the image: the text on the bridge (12), inscriptions (13), and the label on the N Seoul Tower window (14). Where no text was visible, as in the aerial view of NYC (01) and the Miami picture (02), confidence fell to 12-45%, and the agent described its answers as "possibly" or "unverified".

Although the agent managed to identify the location correctly, it couldn't pinpoint the camera location. For example, Photo 15: the text reads Munsusan and the agent reported 99% confidence, yet all three answers were incorrect. Its confidence did not reflect whether its coordinates were within 250 meters of the camera.

The agent also guessed less known places. For the Ulsan lantern festival (08), it read the text on a display and used a whale sculpture to find the city. It used web search in every run, citing sources such as Mapcarta and Wikidata. All 45 Google Maps links matched the stated coordinates.

## Conclusion

ChatGPT's stated confidence is not reliable enough metric to evaluate the correctness of response. Even at a mean stated confidence of 97%, the agent was correct only 71% of the time, and no confidence group came close to its stated level. Moreover, after each run, the confidence score varied (photo 08). At the level of the city and the landmark, however, the agent was consistent and accurate.

I believe it might be better to not use the confidence as the score for correctness. The person should read the clues and reasoning that agent used for answering: some clues are informative whereas other are "bridge" and not that useful information. Based on the agents reasoning the confidence could be understood much better.
