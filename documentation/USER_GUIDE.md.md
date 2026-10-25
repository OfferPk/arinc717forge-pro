# ARINC717Forge Pro User Guide & Developer Manual

## 1\. Quickstart Guide

1. Launch `index.html` in your browser.  
2. In the "Flight Data Recorder Telemetry Slicing" panel, select the Word Rate (e.g. `256 Words/s` standard transport rate).  
3. Set flight telemetry parameters: Altitude (`35,000 ft`), Airspeed (`280 knots`), Pitch (`+2.5°`), and Roll (`0.0°`).  
4. Click **Stream Next Subframe** to assemble the 12-bit telemetry array with the appropriate Barker code (`0x247`, `0x5B8`, `0xA47`, `0xDB8`), advance the 4-second superframe cycle, and view the word grid.  
5. Click **Run Normative Audit** to evaluate all 10 ARINC 717 / FAA Part 121 compliance checks.

## 2\. Standalone Headless Python CLI

Run automated regression testing and compliance verification:

python3 arinc717-cli.py \--audit \--rate 256 \--alt 35000 \--speed 280