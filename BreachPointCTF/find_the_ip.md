# Writeup: Find the IP

**Category:** OSINT **Difficulty:** Easy **Files:** `looks.png`

## Challenge Overview

We are presented with a static image that appears to be a capture from a live webcam feed. The image depicts a cable wakeboarding park with a large artificial lake and solar panels in the foreground. The objective is straight forward: identify the IP address associated with this camera feed.

## 1. Visual Reconnaissance & Reverse Search

The image has a distinct timestamp overlay and a wide-angle field of view characteristic of security or weather cameras. While there is faint text at the bottom ("Hip Notics Web Cam"), the fastest route for unknown locations is a Reverse Image Search.

I passed the image through **Google Lens**.

**Result:** The search engine immediately identified the location as **Hip Notics Cable Park** in **Antalya, Turkey**.

## 2. Tracking the Source

Among the search results, one match pointed to a webcam directory: `https://www.webcamgalore.com/webcam/Turkey/Antalya/584.html`

The thumbnail on this site matched our challenge image perfectly.

## 3. Extracting the Technical Details

To find the IP, I navigated to the "Watch" or live view section of the webcam listing. OSINT challenges involving webcams often rely on the fact that aggregators (like WebcamGalore) frequently link directly to the unproctected IP stream of the camera.

Upon clicking to view the stream, the browser URL bar (or the network inspection tab) revealed the direct address of the camera: `http://78.186.26.188/...`

The IP address `78.186.26.188` is the origin of the feed.

## 4. Flag Construction

Following the standard flag format `BPCTF{IP_ADDRESS}`, I submitted the found IP.

**Final Flag:** `BPCTF{78.186.26.188}`
