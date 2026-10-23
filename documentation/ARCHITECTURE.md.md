\# DALIForge Pro (v1.0.0) — Technical Architecture

\#\# 1\. System Overview  
\*\*DALIForge Pro\*\* is a zero-dependency protocol engineering platform, 16-bit Manchester bi-phase codec, DAPC logarithmic and linear dimming engine, Device Type 8 (DT8) tunable white colour temperature converter, D4i Parts 251–253 energy and diagnostic analyzer, and IEC 62386 smart building cybersecurity auditor for \*\*DALI-2 & D4i (IEC 62386)\*\* commercial lighting systems.

Digital Addressable Lighting Interface (DALI) is the global standard for commercial, institutional, and architectural lighting control. Specified under the international \*\*IEC 62386\*\* multi-part standard and certified by the DALI Alliance (DiiA), DALI controls millions of luminaires across modern office towers, luxury hotels, healthcare centers, transport hubs, and universities.

DALIForge Pro bridges the gap between field electrical commissioning, Building Management System (BMS) integration, and OT cybersecurity operations.

\---

\#\# 2\. Protocol Framing & Physical Layer Architecture

\`\`\`  
\+---------------------------------------------------------------+  
\| Layer 7b: D4i Luminaire Memory Banks (Parts 251 / 252 / 253\)  \|  
\| Bank 201: OEM GTIN/Asset \| Bank 202: Energy Wh \| Bank 203: Diagnostics \|  
\+---------------------------------------------------------------+  
\| Layer 7a: DALI-2 Application Protocol (IEC 62386-102 / 209\)  \|  
\| 16-Bit Forward Frame: \[Address Byte (8b)\] \[Data / Opcode (8b)\] \|  
\| Short (0..63) \| Group (0..15) \| Broadcast (0xFE/0xFF)         \|  
\+---------------------------------------------------------------+  
\| Layer 2: Manchester Bi-Phase Encoding (IEC 62386-101)          \|  
\| 1200 Baud (+/- 10%) \| Bit Period: 833.3 us \| Half-Bit: 416.7 us \|  
\| '1' \= Low-to-High \| '0' \= High-to-Low \| 1 Start \| 2 Stop Bits \|  
\+---------------------------------------------------------------+  
\| Layer 1: Two-Wire Polarity-Free Bus (16V Nominal DC, 250mA Max)\|  
\+---------------------------------------------------------------+  
\`\`\`

\---

\#\# 3\. Addressing & Command Frame Layout

A standard DALI forward frame comprises 16 bits (2 bytes):

\#\#\# 3.1 Address Byte (Byte 0\)  
\- \*\*Bit 0 — Selector Bit (\$S\$)\*\*:  
  \- \`0\`: Direct Arc Power Control (DAPC). Byte 1 represents arc power level (\$0 \\le X \\le 254\$).  
  \- \`1\`: Standard Command. Byte 1 represents command opcode (\$0 \\le X \\le 255\$).  
\- \*\*Bits 7..1 — Target Address\*\*:  
  \- \`0AAAAAA-S\`: Short Address \$0 \\le AAAAAA \\le 63\$.  
  \- \`100AAAA-S\`: Group Address \$0 \\le AAAA \\le 15\$.  
  \- \`1111111-S\`: Broadcast to all control gear (\`0xFE\` for DAPC; \`0xFF\` for Command).  
  \- \`1111110-S\`: Broadcast to unaddressed control gear (\`0xFC\` for DAPC; \`0xFD\` for Command).

\#\#\# 3.2 Special Commands  
Special commands (Opcodes \`0xA1\` through \`0xCB\` with bit 0 set to 1\) perform central commissioning, addressing, and register configuration. Critical safety commands require two identical transmissions within 100 ms:  
\- \`INITIALIZE\` (\`0xA1\`): Unlocks search and addressing registers.  
\- \`RANDOMISE\` (\`0xA3\`): Generates a random 24-bit search address.  
\- \`PROGRAM\_SHORT\_ADDRESS\` (\`0xB7\`): Assigns a short address to the selected gear.  
\- \`ENABLE\_DEVICE\_TYPE\` (\`0xC1\`): Enables device-specific extensions (e.g. Device Type 8 Colour Control).

\---

\#\# 4\. Dimming & Colour Mathematical Formulations

\#\#\# 4.1 Logarithmic Dimming Curve (IEC 62386-102)  
Human visual perception of light intensity is non-linear. The IEC 62386 logarithmic dimming curve calculates physical light output percentage from arc power byte \$X \\in \[1, 254\]\$:  
\$\$\\text{Light Output (\\%)} \= 10^{\\frac{3(X \- 1)}{253} \- 1}\$\$  
\- \$X \= 0\$: Off (\$0\\%\$).  
\- \$X \= 1\$: \$0.1\\%\$ light output (\$10^{-1}\$).  
\- \$X \= 229\$: \$\\approx 50.5\\%\$ physical light output.  
\- \$X \= 254\$: \$100\\%\$ light output (\$10^2\$).

\#\#\# 4.2 Device Type 8 (DT8) Tunable White (IEC 62386-209)  
Colour temperature \$T\_c\$ (Kelvin) is mapped into micro-reciprocal degrees (mirek, \$M\$):  
\$\$M \= \\text{round}\\left( \\frac{10^6}{T\_c} \\right)\$\$  
\- Warm White \$2700\\text{ K} \\rightarrow 370\\text{ mirek}\$.  
\- Neutral White \$4000\\text{ K} \\rightarrow 250\\text{ mirek}\$.  
\- Daylight \$6500\\text{ K} \\rightarrow 154\\text{ mirek}\$.

DALIForge Pro generates the standardized 5-frame commissioning sequence:  
1\. \`ENABLE\_DEVICE\_TYPE 8\` (\`0xC1 0x08\`)  
2\. \`DTR0\` lower mirek byte (\`0xC3 \[M\_low\]\`)  
3\. \`DTR1\` upper mirek byte (\`0xC5 \[M\_high\]\`)  
4\. \`SET\_TEMP\_COLOUR\_TEMP\` (\`\[Addr\] 0xE7\`)  
5\. \`ACTIVATE\` (\`\[Addr\] 0xE0\`)

\---

\#\# 5\. Security & Safety Architecture (IEC 62386 & NFPA 101\)  
1\. \*\*Broadcast Blackout Defense\*\*: Guarding against rogue broadcast \`0xFE 0x00\` (Turn-Off) commands injected over IP/BMS gateways.  
2\. \*\*Emergency Lighting Verification (IEC 62386-202)\*\*: Automated scheduling of monthly functional and annual duration battery discharge tests.  
3\. \*\*Gateway Segmentation\*\*: Isolating DALI-to-BACnet / DALI-to-KNX IP gateways on dedicated building automation VLANs.  
4\. \*\*Runtime Commissioning Lockout\*\*: Preventing unauthorized execution of \`INITIALIZE\` and address remapping outside maintenance windows.  
5\. \*\*D4i Submetering Tamper Protection\*\*: Validating write-protection on Memory Bank 202 energy consumption registers.  
