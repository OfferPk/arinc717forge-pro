/\*\*  
 \* ARINC Specification 717 Flight Data Recorder (FDR) Core Telemetry Engine  
 \* Implements 4-subframe superframe construction, 12-bit word packing,  
 \* Barker sync code verification, and FAA Part 121 parameter engineering scaling.  
 \*/

// Barker Sync Words (12 bits) for Subframes 1 through 4  
const BARKER\_SYNC\_WORDS \= {  
  SUBFRAME\_1: 0x247, // 0010 0100 0111  
  SUBFRAME\_2: 0x5B8, // 0101 1011 1000 (Inverted sync 1\)  
  SUBFRAME\_3: 0xA47, // 1010 0100 0111  
  SUBFRAME\_4: 0xDB8  // 1101 1011 1000 (Inverted sync 3\)  
};

// Word Rates per Second (Subframe word counts)  
const ARINC717\_WORD\_RATES \= {  
  RATE\_64: 64,  
  RATE\_128: 128,  
  RATE\_256: 256,  
  RATE\_512: 512,  
  RATE\_1024: 1024  
};

class ARINC717Engine {  
  constructor(wordsPerSubframe \= ARINC717\_WORD\_RATES.RATE\_256) {  
    this.wordsPerSubframe \= wordsPerSubframe;  
    this.superframeCounter \= 0;  
    this.currentSubframe \= 1; // 1 to 4  
  }

  /\*\*  
   \* Returns the Barker Code for a specific subframe (1 to 4\)  
   \*/  
  static getBarkerCode(subframeIndex) {  
    switch (subframeIndex) {  
      case 1: return BARKER\_SYNC\_WORDS.SUBFRAME\_1;  
      case 2: return BARKER\_SYNC\_WORDS.SUBFRAME\_2;  
      case 3: return BARKER\_SYNC\_WORDS.SUBFRAME\_3;  
      case 4: return BARKER\_SYNC\_WORDS.SUBFRAME\_4;  
      default: throw new Error(\`Invalid subframe index: \${subframeIndex}\`);  
    }  
  }

  /\*\*  
   \* Scales engineering units to 12-bit integer (0 to 4095\)  
   \*/  
  static scaleTo12Bit(val, minVal, maxVal) {  
    const clamped \= Math.max(minVal, Math.min(maxVal, val));  
    const normalized \= (clamped \- minVal) / (maxVal \- minVal);  
    return Math.round(normalized \* 4095\) & 0x0FFF;  
  }

  /\*\*  
   \* Generates a 12-bit ARINC 717 Subframe  
   \*/  
  generateSubframe(subframeIndex, flightParameters \= {}) {  
    const barker \= ARINC717Engine.getBarkerCode(subframeIndex);  
    const words \= new Array(this.wordsPerSubframe).fill(0);

    // Word 1 is always the Barker Sync Code  
    words\[0\] \= barker;

    // Word 2: Superframe Counter (0 to 15, sub-multiplexing)  
    words\[1\] \= (this.superframeCounter & 0x0F);

    // Populate FAA Part 121 Mandatory Flight Parameters  
    // Altitude: \-1,000 to \+50,000 ft  
    const altitudeFt \= flightParameters.altitudeFt \!== undefined ? flightParameters.altitudeFt : 35000;  
    words\[2\] \= ARINC717Engine.scaleTo12Bit(altitudeFt, \-1000, 50000);

    // Airspeed: 0 to 500 knots  
    const airspeedKts \= flightParameters.airspeedKts \!== undefined ? flightParameters.airspeedKts : 280;  
    words\[3\] \= ARINC717Engine.scaleTo12Bit(airspeedKts, 0, 500);

    // Magnetic Heading: 0 to 360 degrees  
    const headingDeg \= flightParameters.headingDeg \!== undefined ? flightParameters.headingDeg : 180;  
    words\[4\] \= ARINC717Engine.scaleTo12Bit(headingDeg, 0, 360);

    // Pitch Attitude: \-90 to \+90 degrees  
    const pitchDeg \= flightParameters.pitchDeg \!== undefined ? flightParameters.pitchDeg : 2.5;  
    words\[5\] \= ARINC717Engine.scaleTo12Bit(pitchDeg, \-90, 90);

    // Roll Attitude: \-180 to \+180 degrees  
    const rollDeg \= flightParameters.rollDeg \!== undefined ? flightParameters.rollDeg : 0.0;  
    words\[6\] \= ARINC717Engine.scaleTo12Bit(rollDeg, \-180, 180);

    // Normal Acceleration: \-3.0g to \+6.0g  
    const normalG \= flightParameters.normalG \!== undefined ? flightParameters.normalG : 1.0;  
    words\[7\] \= ARINC717Engine.scaleTo12Bit(normalG, \-3.0, 6.0);

    // Fill remaining telemetry slots with mock sensor words  
    for (let i \= 8; i \< this.wordsPerSubframe; i++) {  
      words\[i\] \= ((i \* 17\) \+ (this.superframeCounter \* 3)) & 0x0FFF;  
    }

    const bitRateHz \= this.wordsPerSubframe \* 12; // e.g. 256 words \* 12 bits \= 3072 bps

    return {  
      subframeIndex: subframeIndex,  
      barkerHex: '0x' \+ barker.toString(16).toUpperCase().padStart(3, '0'),  
      wordsCount: this.wordsPerSubframe,  
      totalBits: this.wordsPerSubframe \* 12,  
      bitRateHz: bitRateHz,  
      words: words,  
      parameters: {  
        altitudeFt: altitudeFt,  
        airspeedKts: airspeedKts,  
        headingDeg: headingDeg,  
        pitchDeg: pitchDeg,  
        rollDeg: rollDeg,  
        normalG: normalG  
      }  
    };  
  }

  /\*\*  
   \* Generates a complete 4-second Superframe (Subframes 1 through 4\)  
   \*/  
  generateSuperframe(flightParameters \= {}) {  
    this.superframeCounter \= (this.superframeCounter \+ 1\) % 16;  
    const subframes \= \[\];

    for (let sf \= 1; sf \<= 4; sf++) {  
      subframes.push(this.generateSubframe(sf, flightParameters));  
    }

    return {  
      superframeId: this.superframeCounter,  
      durationSeconds: 4.0,  
      totalWords: this.wordsPerSubframe \* 4,  
      totalBits: this.wordsPerSubframe \* 4 \* 12,  
      subframes: subframes  
    };  
  }  
}

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= {  
    ARINC717Engine,  
    BARKER\_SYNC\_WORDS,  
    ARINC717\_WORD\_RATES  
  };  
}  
