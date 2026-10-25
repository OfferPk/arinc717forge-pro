# ARINC Specification 717 Technical Architecture & Specification

## 1\. Executive Summary & Protocol Architecture

ARINC Specification 717 standardizes the flight data acquisition and recording protocol between Digital Flight Data Acquisition Units (DFDAU) and crash-protected Flight Data Recorders (FDR / "Black Box").

### Key Architectural Concepts:

1. **Subframe & Superframe Structure**:  
   - Continuous stream divided into 1-second Subframes (Subframes 1 through 4), repeating every 4.0 seconds as a complete Superframe.  
   - Word rates: 64, 128, 256, 512, or 1024 words per second.  
   - Word length: Exactly 12 bits per telemetry word.  
2. **Barker Code Synchronization (Word 1\)**:  
   - Subframe 1: `0x247` (\$001001000111\_2\$)  
   - Subframe 2: `0x5B8` (\$010110111000\_2\$, Bitwise complement of Subframe 1\)  
   - Subframe 3: `0xA47` (\$101001000111\_2\$)  
   - Subframe 4: `0xDB8` (\$110110111000\_2\$, Bitwise complement of Subframe 3\)  
3. **Mandatory Flight Parameters (FAA 14 CFR Part 121 Appendix M)**:  
   - Word 2: Superframe Counter (4 bits, values 0–15) used for sub-multiplexing slow-changing parameters across multiple superframes.  
   - Core flight telemetry parameters: Pressure Altitude, Calibrated Airspeed, Heading, Pitch/Roll, Normal/Longitudinal Accelerations, Control Surface Positions, and Engine N1/EPR.  
4. **Physical Layer Signaling**:  
   - Bi-Polar Return-to-Zero (BPRZ) or Harvard Biphase self-clocking modulation over shielded twisted pair cable.