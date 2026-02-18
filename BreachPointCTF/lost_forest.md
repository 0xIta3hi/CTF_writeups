# Writeup: Lost Forest

**Category:** OSINT / Forensics **Difficulty:** Easy (with a twist) **Files:** `img.jpg`

## Challenge Overview

We are given an image of a lush, green forest with a running stream. The goal is to identify the location to form the flag. On the surface, this looks like a standard GEOINT (Geospatial Intelligence) challenge requiring visual identification of foliage or terrain. However, the solution lies beneath the surface.

## 1. Metadata Analysis

The first step in any image-based OSINT challenge is to check for hidden metadata. I used `exiftool` to extract the EXIF data from `img.jpg`.

Bash

```
$ exiftool img.jpg
...
GPS Latitude                    : 54 deg 25' 0.00" S
GPS Longitude                   : 3 deg 22' 0.00" E
GPS Position                    : 54 deg 25' 0.00" S, 3 deg 22' 0.00" E
...
```

The image contained explicit GPS coordinates: **54°25'00.0"S 3°22'00.0"E**.

## 2. The Anomaly (Forest vs. Glacier)

Plugging these coordinates into Google Maps leads to **Bouvet Island**, a dependency of **Norway**.

- **The Problem:** Bouvet Island is known as the "most remote island in the world." It is an uninhabited volcanic island in the subantarctic, 93% covered by glaciers. It definitely does **not** have lush green forests or calm flowing brooks like the one in `img.jpg`.
    

## 3. The Rabbit Hole (The "Real" Visual Location)

Suspecting the metadata might be spoofed or corrupted (a common CTF trick), I attempted to "fix" the coordinates by swapping the hemispheres:

- Original: `54°25'S, 3°22'E` (Bouvet Island)
    
- Swapped: `54°25'N, 3°22'W`
    

These swapped coordinates land in the **Lake District, United Kingdom**—a location that perfectly matches the visual evidence of a green, temperate forest.

## 4. The Resolution

I was stuck between two answers:

1. **Visual Truth:** Lake District, UK (Real location of the photo).
    
2. **Digital Truth:** Bouvet Island, Norway (The hardcoded metadata).
    

The challenge organizers released a hint: `{islandname_countryname}`.

This format confirmed that the intended answer relied on the **EXIF data**, regardless of the visual impossibility. The image was likely a stock photo with manually injected coordinates to confuse the player.

**Flag:** `BPCTF{bouvet_island_norway}`

