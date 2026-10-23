\<\!DOCTYPE html\>  
\<html lang="en"\>  
\<head\>  
  \<meta charset="UTF-8"\>  
  \<meta name="viewport" content="width=device-width, initial-scale=1.0"\>  
  \<title\>DALIForge Pro \| Commercial Smart Building DALI-2 \&amp; D4i Protocol Studio\</title\>  
  \<link rel="stylesheet" href="dali-forge.css"\>  
  \<link rel="preconnect" href="https\://fonts.googleapis.com"\>  
  \<link rel="preconnect" href="https\://fonts.gstatic.com" crossorigin\>  
  \<link href="https\://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800\&family=JetBrains+Mono:wght@400;500;600;700\&display=swap" rel="stylesheet"\>  
\</head\>  
\<body\>  
  \<div class="container"\>  
    \<header\>  
      \<div class="brand"\>  
        \<div class="logo-badge"\>DALI\</div\>  
        \<div class="brand-info"\>  
          \<h1\>DALIForge Pro\</h1\>  
          \<p\>Commercial DALI-2 \&amp; D4i (IEC 62386\) Protocol Studio, 16-Bit Manchester Codec \&amp; Cybersecurity Auditor\</p\>  
        \</div\>  
      \</div\>  
      \<div class="header-status"\>  
        \<span class="badge badge-dali"\>IEC 62386-102/209\</span\>  
        \<span class="badge badge-emerald"\>D4i Parts 251-253\</span\>  
        \<span class="badge badge-cyan"\>1200 Baud Manchester\</span\>  
      \</div\>  
    \</header\>

    \<div class="nav-tabs"\>  
      \<button class="tab-btn active" data-tab="tab-dissector"\>Frame Dissector\</button\>  
      \<button class="tab-btn" data-tab="tab-dapc-studio"\>DAPC \&amp; Dimming Studio\</button\>  
      \<button class="tab-btn" data-tab="tab-dt8-d4i"\>DT8 Colour \&amp; D4i Energy\</button\>  
      \<button class="tab-btn" data-tab="tab-auditor"\>IEC 62386 Safety Auditor\</button\>  
      \<button class="tab-btn" data-tab="tab-config"\>Gateway Firewall \&amp; IDS Rules\</button\>  
    \</div\>

    \<\!-- TAB 1: DISSECTOR \--\>  
    \<div id="tab-dissector" class="tab-panel active"\>  
      \<div class="grid-2"\>  
        \<div class="card"\>  
          \<div class="card-title"\>  
            \<span\>Raw DALI-2 16-Bit Frame Input\</span\>  
            \<select id="preset-selector" style="width: auto; padding: 4px 8px; font-size: 0.8rem;"\>  
              \<option value="dapc\_max"\>Preset: Broadcast 100% (FE FE \- DAPC 254)\</option\>  
              \<option value="dapc\_half"\>Preset: Short Addr 0 50% (00 C8 \- DAPC 200)\</option\>  
              \<option value="cmd\_off"\>Preset: Broadcast Turn Off (FF 00 \- Command 0)\</option\>  
              \<option value="cmd\_query"\>Preset: Short Addr 5 Query Status (0B 90 \- Cmd 144)\</option\>  
              \<option value="special\_init"\>Preset: Special Command INITIALIZE (A1 00)\</option\>  
            \</select\>  
          \</div\>  
          \<div class="form-group"\>  
            \<label\>Raw DALI 16-Bit Frame Hex (2 Bytes)\</label\>  
            \<input type="text" id="hex-input" class="mono" value="FE FE" placeholder="e.g. FE FE or 00 C8"\>  
          \</div\>  
          \<div class="btn-group"\>  
            \<button id="btn-dissect" class="btn"\>Dissect Frame\</button\>  
            \<button id="btn-export-pcap" class="btn btn-secondary"\>Export to PCAP\</button\>  
            \<button id="btn-clear-hex" class="btn btn-secondary"\>Clear\</button\>  
          \</div\>  
          \<div style="margin-top: 20px;"\>  
            \<label\>Manchester Bi-Phase Waveform Simulation\</label\>  
            \<div id="manchester-output" class="hex-view" style="max-height: 120px; font-size: 0.75rem;"\>-\</div\>  
          \</div\>  
        \</div\>

        \<div class="card"\>  
          \<div class="card-title"\>Protocol Dissection Tree\</div\>  
          \<div id="dissection-output" class="tree-view"\>  
            \<p style="color: var(--text-muted);"\>Select a preset or enter raw hex and click "Dissect Frame".\</p\>  
          \</div\>  
        \</div\>  
      \</div\>  
    \</div\>

    \<\!-- TAB 2: DAPC STUDIO \--\>  
    \<div id="tab-dapc-studio" class="tab-panel"\>  
      \<div class="grid-2"\>  
        \<div class="card"\>  
          \<div class="card-title"\>Direct Arc Power \&amp; Command Synthesizer\</div\>  
          \<div class="form-group"\>  
            \<label\>Target Addressing Mode\</label\>  
            \<select id="dali-target-type"\>  
              \<option value="broadcast"\>Broadcast (All Control Gear)\</option\>  
              \<option value="short"\>Short Address (Individual Fixture 0-63)\</option\>  
              \<option value="group"\>Group Address (Zone 0-15)\</option\>  
              \<option value="broadcast\_unaddressed"\>Broadcast Unaddressed Only\</option\>  
            \</select\>  
          \</div\>  
          \<div class="form-group" id="group-target-val" style="display: none;"\>  
            \<label id="label-target-val"\>Target Index (0 to 63)\</label\>  
            \<input type="number" id="dali-target-val" value="0" min="0" max="63"\>  
          \</div\>  
          \<div class="form-group"\>  
            \<label\>Operation Mode\</label\>  
            \<select id="dali-mode"\>  
              \<option value="dapc"\>Direct Arc Power Control (DAPC Dimming)\</option\>  
              \<option value="command"\>Standard Command (IEC 62386-102)\</option\>  
            \</select\>  
          \</div\>  
          \<div class="form-group" id="group-dapc-controls"\>  
            \<label\>Arc Power Level (0 \= Off, 1 \= 0.1%, 254 \= 100%, 255 \= Mask)\</label\>  
            \<div style="display:flex; gap:12px; align-items:center;"\>  
              \<input type="range" id="dali-arc-slider" min="0" max="254" value="254" style="flex:1;"\>  
              \<input type="number" id="dali-arc-input" min="0" max="254" value="254" style="width:90px;"\>  
            \</div\>  
            \<div style="margin-top: 8px; font-size: 0.85rem; color: var(--text-muted);"\>  
              Logarithmic Light Output: \<strong id="val-log-percent" style="color:var(--accent-dali);"\>100.0%\</strong\> \| Linear: \<strong id="val-lin-percent" style="color:var(--accent-cyan);"\>100.0%\</strong\>  
            \</div\>  
          \</div\>  
          \<div class="form-group" id="group-cmd-controls" style="display: none;"\>  
            \<label\>Select DALI Standard Command\</label\>  
            \<select id="dali-cmd-select"\>  
              \<option value="0"\>0x00 \- OFF (Turn Off)\</option\>  
              \<option value="1"\>0x01 \- UP (Fade Up 200ms)\</option\>  
              \<option value="2"\>0x02 \- DOWN (Fade Down 200ms)\</option\>  
              \<option value="5"\>0x05 \- RECALL\_MAX\_LEVEL (100%)\</option\>  
              \<option value="6"\>0x06 \- RECALL\_MIN\_LEVEL (Physical Min)\</option\>  
              \<option value="7"\>0x07 \- STEP\_DOWN\_AND\_OFF\</option\>  
              \<option value="8"\>0x08 \- ON\_AND\_STEP\_UP\</option\>  
              \<option value="16"\>0x10 \- GO\_TO\_SCENE\_0\</option\>  
              \<option value="17"\>0x11 \- GO\_TO\_SCENE\_1\</option\>  
              \<option value="32"\>0x20 \- RESET\</option\>  
              \<option value="144"\>0x90 \- QUERY\_STATUS\</option\>  
              \<option value="146"\>0x92 \- QUERY\_LAMP\_FAILURE\</option\>  
              \<option value="160"\>0xA0 \- QUERY\_ACTUAL\_LEVEL\</option\>  
            \</select\>  
          \</div\>  
          \<div class="btn-group"\>  
            \<button id="btn-build-frame" class="btn"\>Synthesize DALI Frame\</button\>  
          \</div\>  
        \</div\>

        \<div class="card"\>  
          \<div class="card-title"\>Synthesized Wire Representation\</div\>  
          \<label\>16-Bit DALI Forward Frame Hex\</label\>  
          \<div id="synth-hex-output" class="hex-view"\>Click "Synthesize DALI Frame" to generate.\</div\>  
          \<div style="margin-top: 16px;"\>  
            \<label\>Logarithmic Dimming Formula Breakdown\</label\>  
            \<div id="synth-breakdown" class="tree-view" style="max-height: 180px;"\>-\</div\>  
          \</div\>  
        \</div\>  
      \</div\>  
    \</div\>

    \<\!-- TAB 3: DT8 & D4I \--\>  
    \<div id="tab-dt8-d4i" class="tab-panel"\>  
      \<div class="grid-2"\>  
        \<div class="card"\>  
          \<div class="card-title"\>DT8 Tunable White \&amp; Circadian Lighting\</div\>  
          \<p style="color: var(--text-muted); font-size: 0.85rem; margin-bottom: 14px;"\>  
            IEC 62386-209 Colour Temperature (Tc) converted to Mirek (\$M \= 10^6 / K\$).  
          \</p\>  
          \<div class="form-group"\>  
            \<label\>Color Temperature (Kelvin: 2000K to 6500K)\</label\>  
            \<div style="display:flex; gap:12px; align-items:center;"\>  
              \<input type="range" id="dt8-kelvin-slider" min="2000" max="6500" value="4000" step="50" style="flex:1;"\>  
              \<input type="number" id="dt8-kelvin-input" min="2000" max="6500" value="4000" style="width:100px;"\>  
            \</div\>  
            \<div style="margin-top: 8px; font-size: 0.85rem; color: var(--text-muted);"\>  
              Calculated Mirek: \<strong id="dt8-mirek-val" style="color:var(--accent-dali);"\>250 mirek\</strong\>  
            \</div\>  
          \</div\>  
          \<div class="btn-group"\>  
            \<button id="btn-build-dt8" class="btn"\>Generate DT8 Command Sequence\</button\>  
          \</div\>  
          \<div style="margin-top: 16px;"\>  
            \<label\>Generated 5-Step DALI Sequence\</label\>  
            \<div id="dt8-seq-output" class="hex-view" style="max-height: 140px;"\>-\</div\>  
          \</div\>  
        \</div\>

        \<div class="card"\>  
          \<div class="card-title"\>D4i Luminaire Energy Telemetry (Part 252)\</div\>  
          \<p style="color: var(--text-muted); font-size: 0.85rem; margin-bottom: 14px;"\>  
            Memory Bank 202 Energy and Power telemetry registers parser.  
          \</p\>  
          \<div class="form-group"\>  
            \<label\>Simulated Memory Bank 202 Data (12 Bytes Hex)\</label\>  
            \<input type="text" id="d4i-hex-input" class="mono" value="00 00 00 01 86 A0 00 32 FE 00 00 00"\>  
            \<small style="color: var(--text-muted);"\>Active Energy (6B) \+ Active Power (2B) \+ Power Factor (1B)\</small\>  
          \</div\>  
          \<div class="btn-group"\>  
            \<button id="btn-parse-d4i" class="btn btn-secondary"\>Parse D4i Energy\</button\>  
          \</div\>  
          \<div style="margin-top: 16px;"\>  
            \<div id="d4i-output" class="tree-view" style="max-height: 180px;"\>-\</div\>  
          \</div\>  
        \</div\>  
      \</div\>  
    \</div\>

    \<\!-- TAB 4: AUDITOR \--\>  
    \<div id="tab-auditor" class="tab-panel"\>  
      \<div class="card"\>  
        \<div class="card-title"\>  
          \<span\>IEC 62386 \&amp; Building Safety Security Audit\</span\>  
          \<button id="btn-run-audit" class="btn" style="padding: 6px 14px;"\>Run Safety Audit\</button\>  
        \</div\>  
        \<div class="score-display"\>  
          \<div\>  
            \<div class="score-num" id="audit-score"\>72\</div\>  
            \<div style="font-size: 0.8rem; color: var(--text-muted);"\>SAFETY SCORE\</div\>  
          \</div\>  
          \<div\>  
            \<div style="font-size: 1.2rem; font-weight: 700;" id="audit-posture"\>MODERATE POSTURE\</div\>  
            \<p style="color: var(--text-muted); font-size: 0.85rem;" id="audit-summary"\>7 of 10 lighting cybersecurity \&amp; safety controls currently pass criteria.\</p\>  
          \</div\>  
        \</div\>

        \<div style="overflow-x: auto;"\>  
          \<table class="audit-table"\>  
            \<thead\>  
              \<tr\>  
                \<th\>Rule ID\</th\>  
                \<th\>Safety \&amp; Security Control Title\</th\>  
                \<th\>Standard\</th\>  
                \<th\>Severity\</th\>  
                \<th\>Status\</th\>  
                \<th\>Remediation Strategy\</th\>  
              \</tr\>  
            \</thead\>  
            \<tbody id="audit-tbody"\>  
              \<\!-- Rendered via JS \--\>  
            \</tbody\>  
          \</table\>  
        \</div\>  
      \</div\>  
    \</div\>

    \<\!-- TAB 5: CONFIG SYNTHESIZER \--\>  
    \<div id="tab-config" class="tab-panel"\>  
      \<div class="grid-2"\>  
        \<div class="card"\>  
          \<div class="card-title"\>Linux nftables Lighting Conduit Firewall\</div\>  
          \<p style="color: var(--text-muted); font-size: 0.85rem; margin-bottom: 12px;"\>Conduit filter protecting DALI-over-IP (51820) and BACnet/IP (47808) interfaces.\</p\>  
          \<div id="nftables-output" class="hex-view" style="max-height: 380px;"\>\</div\>  
        \</div\>  
        \<div class="card"\>  
          \<div class="card-title"\>Suricata L7 Industrial IDS Rules\</div\>  
          \<p style="color: var(--text-muted); font-size: 0.85rem; margin-bottom: 12px;"\>Signatures monitoring broadcast blackout injection and stroboscopic cycling.\</p\>  
          \<div id="suricata-output" class="hex-view" style="max-height: 380px;"\>\</div\>  
        \</div\>  
      \</div\>  
    \</div\>

  \</div\>

  \<script src="dali-protocol-engine.js"\>\</script\>  
  \<script src="dali-security-auditor.js"\>\</script\>  
  \<script src="dali-config-synthesizer.js"\>\</script\>  
  \<script src="app.js"\>\</script\>  
\</body\>  
\</html\>  
