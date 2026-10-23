/\*\*  
 \* DALIForge Pro \- DALI-2 & D4i (IEC 62386\) Protocol Engine  
 \* Standards: IEC 62386-101/102/103/207/209, D4i Parts 251/252/253, DiiA Specifications  
 \* Author: Autonomous Digital Product Factory (Product \#171)  
 \* Version: 1.0.0  
 \*/

class DALIEngine {  
  constructor() {  
    this.BAUD\_RATE \= 1200;  
    this.BIT\_PERIOD\_US \= 833.33; // 1/1200 s in microseconds  
    this.HALF\_BIT\_US \= 416.67;

    this.COMMANDS \= {  
      0x00: 'OFF',  
      0x01: 'UP',  
      0x02: 'DOWN',  
      0x03: 'STEP\_UP',  
      0x04: 'STEP\_DOWN',  
      0x05: 'RECALL\_MAX\_LEVEL',  
      0x06: 'RECALL\_MIN\_LEVEL',  
      0x07: 'STEP\_DOWN\_AND\_OFF',  
      0x08: 'ON\_AND\_STEP\_UP',  
      0x0A: 'DAPC\_STEP\_DOWN\_AND\_OFF',  
      0x0B: 'DAPC\_ON\_AND\_STEP\_UP',  
      0x20: 'RESET',  
      0x21: 'STORE\_ACTUAL\_LEVEL\_IN\_DTR0',  
      0x2A: 'STORE\_DTR0\_AS\_MAX\_LEVEL',  
      0x2B: 'STORE\_DTR0\_AS\_MIN\_LEVEL',  
      0x2C: 'STORE\_DTR0\_AS\_SYSTEM\_FAILURE\_LEVEL',  
      0x2D: 'STORE\_DTR0\_AS\_POWER\_ON\_LEVEL',  
      0x2E: 'STORE\_DTR0\_AS\_FADE\_TIME',  
      0x2F: 'STORE\_DTR0\_AS\_FADE\_RATE',  
      0x90: 'QUERY\_STATUS',  
      0x91: 'QUERY\_CONTROL\_GEAR\_PRESENT',  
      0x92: 'QUERY\_LAMP\_FAILURE',  
      0x93: 'QUERY\_LAMP\_POWER\_ON',  
      0x94: 'QUERY\_LIMIT\_ERROR',  
      0x95: 'QUERY\_RESET\_STATE',  
      0x96: 'QUERY\_MISSING\_SHORT\_ADDRESS',  
      0x97: 'QUERY\_VERSION\_NUMBER',  
      0x98: 'QUERY\_CONTENT\_DTR0',  
      0x99: 'QUERY\_DEVICE\_TYPE',  
      0x9A: 'QUERY\_PHYSICAL\_MIN\_LEVEL',  
      0xA0: 'QUERY\_ACTUAL\_LEVEL',  
      0xA1: 'QUERY\_MAX\_LEVEL',  
      0xA2: 'QUERY\_MIN\_LEVEL',  
      0xA3: 'QUERY\_POWER\_ON\_LEVEL',  
      0xA4: 'QUERY\_SYSTEM\_FAILURE\_LEVEL',  
      0xA5: 'QUERY\_FADE\_TIME\_AND\_FADE\_RATE'  
    };

    this.SPECIAL\_COMMANDS \= {  
      0xA1: 'INITIALIZE',  
      0xA3: 'RANDOMISE',  
      0xA5: 'COMPARE',  
      0xA7: 'WITHDRAW',  
      0xA9: 'PING',  
      0xB1: 'SEARCHADDRH',  
      0xB3: 'SEARCHADDRM',  
      0xB5: 'SEARCHADDRL',  
      0xB7: 'PROGRAM\_SHORT\_ADDRESS',  
      0xB9: 'VERIFY\_SHORT\_ADDRESS',  
      0xBB: 'QUERY\_SHORT\_ADDRESS',  
      0xC1: 'ENABLE\_DEVICE\_TYPE',  
      0xC3: 'DATA\_TRANSFER\_REGISTER\_0 (DTR0)',  
      0xC5: 'DATA\_TRANSFER\_REGISTER\_1 (DTR1)',  
      0xC7: 'DATA\_TRANSFER\_REGISTER\_2 (DTR2)',  
      0xC9: 'WRITE\_MEMORY\_LOCATION',  
      0xCB: 'WRITE\_MEMORY\_LOCATION\_NO\_REPLY'  
    };  
  }

  // \--- DALI Dimming Curve Calculations (IEC 62386-102) \---  
  dapcLevelToPercentLog(level) {  
    if (level \=== 0\) return 0.0;  
    if (level \=== 255\) return null; // Mask  
    // Formula: Light Output (%) \= 10^( (3 \* (X \- 1\) / 253\) \- 1 )  
    const exponent \= (3 \* (level \- 1\) / 253\) \- 1;  
    return Math.pow(10, exponent);  
  }

  percentToDapcLevelLog(percent) {  
    if (percent \<= 0\) return 0;  
    if (percent \> 100\) percent \= 100;  
    // Inverse: X \= 1 \+ (253 / 3\) \* (log10(percent) \+ 1\)  
    const logVal \= Math.log10(percent);  
    const level \= 1 \+ (253 / 3\) \* (logVal \+ 1);  
    return Math.min(254, Math.max(1, Math.round(level)));  
  }

  dapcLevelToPercentLinear(level) {  
    if (level \=== 0\) return 0.0;  
    if (level \=== 255\) return null;  
    return ((level \- 1\) / 253\) \* 100.0;  
  }

  percentToDapcLevelLinear(percent) {  
    if (percent \<= 0\) return 0;  
    if (percent \> 100\) percent \= 100;  
    return Math.min(254, Math.max(1, Math.round(1 \+ (percent / 100.0) \* 253)));  
  }

  // \--- DT8 Tunable White Calculations (IEC 62386-209) \---  
  kelvinToMirek(kelvin) {  
    if (kelvin \<= 0\) return 0;  
    return Math.round(1000000 / kelvin);  
  }

  mirekToKelvin(mirek) {  
    if (mirek \<= 0\) return 0;  
    return Math.round(1000000 / mirek);  
  }

  // \--- Address Encoding Helper \---  
  encodeAddressByte(targetType, targetValue, isCommand \= false) {  
    let addrByte \= 0;  
    const sBit \= isCommand ? 0x01 : 0x00; // Bit 0: 0=DAPC, 1=Command

    if (targetType \=== 'short') {  
      const shortAddr \= Math.min(63, Math.max(0, targetValue));  
      addrByte \= (shortAddr \<\< 1\) \| sBit; // 0AAAAAA-S  
    } else if (targetType \=== 'group') {  
      const groupAddr \= Math.min(15, Math.max(0, targetValue));  
      addrByte \= 0x80 \| (groupAddr \<\< 1\) \| sBit; // 100AAA0-S  
    } else if (targetType \=== 'broadcast\_unaddressed') {  
      addrByte \= 0xFC \| sBit; // 1111110-S  
    } else { // broadcast  
      addrByte \= 0xFE \| sBit; // 1111111-S  
    }  
    return addrByte;  
  }

  // \--- 16-Bit Forward Frame Builder \---  
  buildDapcFrame(targetType, targetValue, level) {  
    const addr \= this.encodeAddressByte(targetType, targetValue, false);  
    const data \= Math.min(255, Math.max(0, level));  
    const frame \= new Uint8Array(\[addr, data\]);  
    return {  
      rawBytes: frame,  
      hexString: this.bytesToHex(frame),  
      type: 'DAPC',  
      level,  
      percentLog: this.dapcLevelToPercentLog(level),  
      percentLinear: this.dapcLevelToPercentLinear(level)  
    };  
  }

  buildCommandFrame(targetType, targetValue, commandOpcode) {  
    const addr \= this.encodeAddressByte(targetType, targetValue, true);  
    const opcode \= Math.min(255, Math.max(0, commandOpcode));  
    const frame \= new Uint8Array(\[addr, opcode\]);  
    return {  
      rawBytes: frame,  
      hexString: this.bytesToHex(frame),  
      type: 'COMMAND',  
      commandName: this.COMMANDS\[opcode\] \|\| \`Command 0x\${opcode.toString(16)}\`,  
      opcode  
    };  
  }

  buildSpecialCommandFrame(specialOpcode, dataByte) {  
    const frame \= new Uint8Array(\[specialOpcode, dataByte\]);  
    return {  
      rawBytes: frame,  
      hexString: this.bytesToHex(frame),  
      type: 'SPECIAL\_COMMAND',  
      commandName: this.SPECIAL\_COMMANDS\[specialOpcode\] \|\| \`Special 0x\${specialOpcode.toString(16)}\`,  
      opcode: specialOpcode,  
      data: dataByte  
    };  
  }

  // \--- DT8 Tunable White Sequence Generator \---  
  buildDt8ColorTempSequence(targetType, targetValue, kelvin) {  
    const mirek \= this.kelvinToMirek(kelvin);  
    const mirekLow \= mirek & 0xFF;  
    const mirekHigh \= (mirek \>\> 8\) & 0xFF;

    const frames \= \[  
      this.buildSpecialCommandFrame(0xC1, 8),          // ENABLE\_DEVICE\_TYPE 8 (DT8)  
      this.buildSpecialCommandFrame(0xC3, mirekLow),   // DTR0 \= Mirek Low  
      this.buildSpecialCommandFrame(0xC5, mirekHigh),  // DTR1 \= Mirek High  
      this.buildCommandFrame(targetType, targetValue, 0xE7), // SET\_TEMP\_COLOUR\_TEMP (Cmd 231\)  
      this.buildCommandFrame(targetType, targetValue, 0xE0)  // ACTIVATE (Cmd 224\)  
    \];

    return {  
      kelvin,  
      mirek,  
      frames,  
      hexSequence: frames.map(f \=\> f.hexString).join(' \| ')  
    };  
  }

  // \--- Manchester Bi-Phase Encoding Simulator \---  
  encodeManchester(frameBytes) {  
    // 16-bit DALI frame: Start bit '1', 16 data bits (MSB first), 2 stop bits (idle high)  
    // In DALI Manchester: '1' is low-then-high (01); '0' is high-then-low (10)  
    let bitStream \= '1'; // Start bit is logical '1'  
    for (let b \= 0; b \< frameBytes.length; b++) {  
      for (let i \= 7; i \>= 0; i--) {  
        bitStream \+= ((frameBytes\[b\] \>\> i) & 1\) ? '1' : '0';  
      }  
    }

    let manchesterWave \= '';  
    for (let i \= 0; i \< bitStream.length; i++) {  
      manchesterWave \+= (bitStream\[i\] \=== '1') ? '01' : '10';  
    }  
    // Append 2 stop bits (idle high)  
    manchesterWave \+= '1111';

    return {  
      bitStream,  
      manchesterHalfBits: manchesterWave,  
      durationUs: (bitStream.length \+ 2\) \* this.BIT\_PERIOD\_US  
    };  
  }

  // \--- Dissector \---  
  dissect(bytes) {  
    if (bytes.length \< 2\) {  
      throw new Error('Frame too short for DALI-2 16-bit Forward Frame (\< 2 bytes)');  
    }

    const b0 \= bytes\[0\];  
    const b1 \= bytes\[1\];

    const isSpecial \= (b0 \>= 0xA1 && b0 \<= 0xCB && (b0 & 1\) \=== 1);

    if (isSpecial) {  
      return {  
        frameType: 'Special Command (IEC 62386-102)',  
        specialOpcode: \`0x\${b0.toString(16).toUpperCase()}\`,  
        specialName: this.SPECIAL\_COMMANDS\[b0\] \|\| 'Unknown Special Command',  
        dataByte: \`0x\${b1.toString(16).toUpperCase()} (\${b1})\`,  
        requiresTwice: \['INITIALIZE', 'RANDOMISE', 'PROGRAM\_SHORT\_ADDRESS'\].some(n \=\> (this.SPECIAL\_COMMANDS\[b0\]\|\|'').includes(n))  
      };  
    }

    const isCommand \= (b0 & 0x01) \=== 1;  
    let targetType \= 'Short Address';  
    let targetIndex \= (b0 \>\> 1\) & 0x3F;  
    let targetDesc \= \`Short Address \${targetIndex} (A\${targetIndex})\`;

    if ((b0 & 0x80) \!== 0\) {  
      if ((b0 & 0xFE) \=== 0xFE) {  
        targetType \= 'Broadcast';  
        targetDesc \= 'Broadcast to All Gear';  
      } else if ((b0 & 0xFE) \=== 0xFC) {  
        targetType \= 'Broadcast Unaddressed';  
        targetDesc \= 'Broadcast to Unaddressed Gear Only';  
      } else {  
        targetType \= 'Group Address';  
        targetIndex \= (b0 \>\> 1\) & 0x0F;  
        targetDesc \= \`Group Address \${targetIndex} (G\${targetIndex})\`;  
      }  
    }

    const res \= {  
      addressing: {  
        raw: \`0x\${b0.toString(16).toUpperCase()}\`,  
        targetType,  
        targetDesc,  
        selectorBit: isCommand ? '1 (Standard Command)' : '0 (Direct Arc Power Control)'  
      }  
    };

    if (\!isCommand) {  
      // DAPC  
      const level \= b1;  
      res.dapc \= {  
        level,  
        hex: \`0x\${level.toString(16).toUpperCase()}\`,  
        percentLog: level \=== 0 ? '0% (OFF)' : (level \=== 255 ? 'MASK' : \`\${this.dapcLevelToPercentLog(level).toFixed(2)}%\`),  
        percentLinear: level \=== 0 ? '0% (OFF)' : (level \=== 255 ? 'MASK' : \`\${this.dapcLevelToPercentLinear(level).toFixed(2)}%\`)  
      };  
    } else {  
      // Command  
      const opcode \= b1;  
      let cmdName \= this.COMMANDS\[opcode\] \|\| 'Vendor Specific';  
      if (opcode \>= 0x10 && opcode \<= 0x1F) {  
        cmdName \= \`GO\_TO\_SCENE\_\${opcode \- 0x10}\`;  
      }  
      res.command \= {  
        opcode: \`0x\${opcode.toString(16).toUpperCase()}\`,  
        commandNumber: opcode,  
        commandName,  
        isConfigCommand: (opcode \>= 0x20 && opcode \<= 0x2F),  
        isQueryCommand: (opcode \>= 0x90 && opcode \<= 0xAF)  
      };  
    }

    return res;  
  }

  // \--- D4i Energy & Diagnostic Data Parser (Parts 251 / 252 / 253\) \---  
  parseD4iEnergyBank(bankBytes) {  
    if (bankBytes.length \< 12\) {  
      return { valid: false, error: 'Bank 202 requires at least 12 bytes' };  
    }  
    // Part 252: Active Energy (6 bytes, Wh) \+ Active Power (2 bytes, W)  
    let energyWh \= 0;  
    for (let i \= 0; i \< 6; i++) {  
      energyWh \= (energyWh \* 256\) \+ bankBytes\[i\];  
    }  
    const powerW \= (bankBytes\[6\] \<\< 8\) \| bankBytes\[7\];  
    const powerFactor \= bankBytes\[8\] / 255.0;

    return {  
      valid: true,  
      standard: 'D4i Part 252 (Energy Reporting)',  
      activeEnergyWh: energyWh,  
      activeEnergyKWh: (energyWh / 1000.0).toFixed(3),  
      activePowerWatts: powerW,  
      powerFactor: powerFactor.toFixed(2)  
    };  
  }

  // \--- Wireshark Libpcap 2.4 Exporter \---  
  exportPcap(packets) {  
    const PCAP\_MAGIC \= 0xA1B2C3D4;  
    const PCAP\_SNAPLEN \= 65535;  
    const LINKTYPE\_ETHERNET \= 1;

    let totalSize \= 24;  
    for (const pkt of packets) {  
      const frameLen \= 14 \+ 20 \+ 8 \+ pkt.length;  
      totalSize \+= 16 \+ frameLen;  
    }

    const buffer \= new ArrayBuffer(totalSize);  
    const view \= new DataView(buffer);  
    const bytes \= new Uint8Array(buffer);

    view.setUint32(0, PCAP\_MAGIC, true);  
    view.setUint16(4, 2, true);  
    view.setUint16(6, 4, true);  
    view.setInt32(8, 0, true);  
    view.setUint32(12, 0, true);  
    view.setUint32(16, PCAP\_SNAPLEN, true);  
    view.setUint32(20, LINKTYPE\_ETHERNET, true);

    let offset \= 24;  
    const baseTime \= Math.floor(Date.now() / 1000);

    packets.forEach((pkt, idx) \=\> {  
      const frameLen \= 14 \+ 20 \+ 8 \+ pkt.length;  
      view.setUint32(offset, baseTime \+ idx, true);  
      view.setUint32(offset \+ 4, idx \* 50000, true);  
      view.setUint32(offset \+ 8, frameLen, true);  
      view.setUint32(offset \+ 12, frameLen, true);  
      offset \+= 16;

      const eth \= \[0x00, 0x50, 0xc2, 0x12, 0x34, 0x56, 0x00, 0x0e, 0x8c, 0xab, 0xcd, 0xef, 0x08, 0x00\];  
      bytes.set(eth, offset);  
      offset \+= 14;

      const ipLen \= 20 \+ 8 \+ pkt.length;  
      const ip \= \[  
        0x45, 0x00, (ipLen \>\> 8\) & 0xff, ipLen & 0xff,  
        0x12, 0x34, 0x40, 0x00,  
        0x40, 0x11, 0x00, 0x00,  
        192, 168, 1, 100, // BMS Controller  
        192, 168, 1, 80   // DALI IP Gateway  
      \];  
      bytes.set(ip, offset);  
      offset \+= 20;

      const udpLen \= 8 \+ pkt.length;  
      const udp \= \[  
        (51820 \>\> 8\) & 0xff, 51820 & 0xff,  
        (51820 \>\> 8\) & 0xff, 51820 & 0xff,  
        (udpLen \>\> 8\) & 0xff, udpLen & 0xff,  
        0x00, 0x00  
      \];  
      bytes.set(udp, offset);  
      offset \+= 8;

      bytes.set(pkt, offset);  
      offset \+= pkt.length;  
    });

    return bytes;  
  }

  // \--- Utilities \---  
  hexToBytes(hex) {  
    const clean \= hex.replace(/\[\\s\\-\_:\\n\\r\]/g, '');  
    if (clean.length % 2 \!== 0\) throw new Error('Invalid hex string length');  
    const bytes \= new Uint8Array(clean.length / 2);  
    for (let i \= 0; i \< clean.length; i \+= 2\) {  
      bytes\[i / 2\] \= parseInt(clean.substr(i, 2), 16);  
    }  
    return bytes;  
  }

  bytesToHex(bytes) {  
    return Array.from(bytes).map(b \=\> b.toString(16).padStart(2, '0')).join(' ');  
  }  
}

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= { DALIEngine };  
}  
