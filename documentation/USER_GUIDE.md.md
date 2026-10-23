\# DALIForge Pro (v1.0.0) — Operator & Engineer User Guide

\#\# 1\. Quick Start Guide

\#\#\# 1.1 Web Application Studio  
Open \`source-code/index.html\` in any modern web browser. Completely standalone with zero backend dependencies.  
\- \*\*Tab 1: Frame Dissector\*\*: Dissect raw 16-bit DALI forward frames, inspect addressing bits, verify DAPC levels and command opcodes, and simulate Manchester waveforms.  
\- \*\*Tab 2: DAPC & Dimming Studio\*\*: Synthesize Direct Arc Power Control (DAPC) or standard command frames for short, group, or broadcast targets with live logarithmic and linear light output calculations.  
\- \*\*Tab 3: DT8 Colour & D4i Energy\*\*: Convert Kelvin to mirek and generate 5-frame tunable white sequences; parse D4i Memory Bank 202 energy telemetry.  
\- \*\*Tab 4: IEC 62386 Safety Auditor\*\*: Evaluate commercial smart lighting installations across 10 safety and cybersecurity criteria.  
\- \*\*Tab 5: Gateway Firewall & IDS Rules\*\*: Generate Linux \`nftables\` lighting conduit rules and Suricata Layer 7 intrusion detection signatures.

\#\#\# 1.2 Standalone Python CLI (\`daliforge-cli.py\`)  
Run directly from terminal using Python 3.8+.

\`\`\`bash  
\# Make executable  
chmod \+x build/daliforge-cli.py

\# 1\. Synthesize Broadcast DAPC Maximum Level (100% / Level 254\)  
./build/daliforge-cli.py dapc \--target broadcast \--level 254

\# 2\. Synthesize Short Address 0 DAPC 50% Level (Level 229\)  
./build/daliforge-cli.py dapc \--target short \--index 0 \--level 229

\# 3\. Synthesize Broadcast Turn-Off Command (0x00)  
./build/daliforge-cli.py cmd \--target broadcast \--opcode 0x00

\# 4\. Synthesize Short Address 5 RECALL\_MAX\_LEVEL (0x05)  
./build/daliforge-cli.py cmd \--target short \--index 5 \--opcode 0x05

\# 5\. Generate DT8 Tunable White 5-Frame Sequence for 4000K (250 mirek)  
./build/daliforge-cli.py dt8-tw \--kelvin 4000

\# 6\. Dissect raw 16-bit DALI frame hex  
./build/daliforge-cli.py dissect "0B 90"

\# 7\. Parse D4i Memory Bank 202 Energy Telemetry Bytes  
./build/daliforge-cli.py parse-d4i "00 00 00 01 86 A0 00 32 FE"

\# 8\. Run IEC 62386 Smart Lighting Safety & Cybersecurity Audit  
./build/daliforge-cli.py audit

\# 9\. Synthesize nftables conduit rules and Suricata signatures  
./build/daliforge-cli.py synthesize-firewall \--bms-ip 192.168.20.10

\# 10\. Export sample DALI sequence to Wireshark-compatible PCAP  
./build/daliforge-cli.py export-pcap \--out dali\_sample.pcap  
\`\`\`

\---

\#\# 2\. CLI Command Reference

\#\#\# \`dapc\`  
Constructs a 16-bit Direct Arc Power Control (DAPC) frame.  
\- \`--target\`: Target addressing (\`broadcast\`, \`short\`, \`group\`). Default: \`broadcast\`.  
\- \`--index\`: Short address (\$0 \\le \\text{index} \\le 63\$) or Group address (\$0 \\le \\text{index} \\le 15\$). Default: \`0\`.  
\- \`--level\`: Arc power level integer (\$0 \\le \\text{level} \\le 254\$). Default: \`254\`.

\#\#\# \`dt8-tw\`  
Synthesizes the complete 5-step DALI-2 Device Type 8 (Colour Control) Tunable White sequence.  
\- \`--kelvin\`: Target color temperature in Kelvin (\$2000 \\le \\text{kelvin} \\le 6500\$). Default: \`4000\`.

\#\#\# \`parse-d4i\`  
Decodes Memory Bank 202 (Energy Reporting) registers, extracting cumulative active energy (Wh / kWh), active power (Watts), and power factor.

\---

\#\# 3\. Engineering Reference & FAQs

\*\*Q: Why does DALI use logarithmic dimming by default?\*\*  
A: Human eyes perceive light logarithmically. At 1% electrical power output, human vision perceives the light at approximately 10% brightness. The IEC 62386 logarithmic curve aligns physical arc levels with human perceptual comfort.

\*\*Q: What is the physical bus voltage of DALI?\*\*  
A: Nominal 16V DC (operating window: 11.5V to 20.5V). Maximum bus current is clamped at 250 mA across a maximum of 64 control gear per loop.  
