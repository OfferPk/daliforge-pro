/\*\*  
 \* DALIForge Pro \- Web Application Controller  
 \*/

document.addEventListener('DOMContentLoaded', () \=\> {  
  const engine \= new DALIEngine();  
  const auditor \= new DALISecurityAuditor();  
  const synthesizer \= new DALIConfigSynthesizer();

  // Presets  
  const PRESETS \= {  
    dapc\_max: "FE FE", // Broadcast 254 (100%)  
    dapc\_half: "00 C8", // Short 0, DAPC 200 (\~50%)  
    cmd\_off: "FF 00", // Broadcast Command 0 (OFF)  
    cmd\_query: "0B 90", // Short 5 (0b0000101-1), Query Status  
    special\_init: "A1 00" // Special INITIALIZE  
  };

  // Tab switching  
  const tabBtns \= document.querySelectorAll('.tab-btn');  
  const tabPanels \= document.querySelectorAll('.tab-panel');

  tabBtns.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      tabBtns.forEach(b \=\> b.classList.remove('active'));  
      tabPanels.forEach(p \=\> p.classList.remove('active'));  
      btn.classList.add('active');  
      const target \= document.getElementById(btn.getAttribute('data-tab'));  
      if (target) target.classList.add('active');  
    });  
  });

  // Preset selector  
  const presetSelector \= document.getElementById('preset-selector');  
  const hexInput \= document.getElementById('hex-input');

  presetSelector.addEventListener('change', () \=\> {  
    const val \= presetSelector.value;  
    if (PRESETS\[val\]) {  
      hexInput.value \= PRESETS\[val\];  
      runDissection();  
    }  
  });

  // Dissector button  
  document.getElementById('btn-dissect').addEventListener('click', runDissection);  
  document.getElementById('btn-clear-hex').addEventListener('click', () \=\> {  
    hexInput.value \= '';  
    document.getElementById('dissection-output').innerHTML \= '\<p style="color: var(--text-muted);"\>Cleared.\</p\>';  
    document.getElementById('manchester-output').innerText \= '-';  
  });

  function runDissection() {  
    const hex \= hexInput.value.trim();  
    if (\!hex) return;  
    const output \= document.getElementById('dissection-output');  
    try {  
      const bytes \= engine.hexToBytes(hex);  
      const res \= engine.dissect(bytes);

      let html \= '';  
      if (res.frameType) { // Special command  
        html \= \`  
          \<div class="tree-node"\>  
            \<span class="tree-key"\>Frame Category:\</span\> \<span class="tree-val" style="color:var(--accent-dali); font-weight:700;"\>\${res.frameType}\</span\>\<br\>  
            \&nbsp;\&nbsp;Opcode: \<span class="tree-val"\>\${res.specialOpcode}\</span\> \| Command: \<span class="tree-val" style="font-weight:700; color:\#38bdf8;"\>\${res.specialName}\</span\>\<br\>  
            \&nbsp;\&nbsp;Data Value: \<span class="tree-val"\>\${res.dataByte}\</span\>\<br\>  
            \&nbsp;\&nbsp;Safety Precondition: \<span class="tree-val" style="color:\${res.requiresTwice ? '\#f43f5e' : '\#10b981'}; font-weight:600;"\>\${res.requiresTwice ? 'Requires Repeat within 100ms' : 'Single Transmission'}\</span\>  
          \</div\>  
        \`;  
      } else {  
        html \= \`  
          \<div class="tree-node"\>  
            \<span class="tree-key"\>Address Byte Decoding (IEC 62386-102):\</span\>\<br\>  
            \&nbsp;\&nbsp;Raw: \<span class="tree-val"\>\${res.addressing.raw}\</span\> \| Target: \<span class="tree-val" style="color:\#06b6d4; font-weight:700;"\>\${res.addressing.targetDesc}\</span\>\<br\>  
            \&nbsp;\&nbsp;Selector Bit (S): \<span class="tree-val"\>\${res.addressing.selectorBit}\</span\>  
          \</div\>  
        \`;

        if (res.dapc) {  
          html \+= \`  
            \<div class="tree-node"\>  
              \<span class="tree-key"\>Direct Arc Power Control (DAPC):\</span\>\<br\>  
              \&nbsp;\&nbsp;Level Byte: \<span class="tree-val"\>\${res.dapc.level} (\${res.dapc.hex})\</span\>\<br\>  
              \&nbsp;\&nbsp;Logarithmic Light Output: \<span class="tree-val" style="color:var(--accent-dali); font-size:1.1rem; font-weight:800;"\>\${res.dapc.percentLog}\</span\>\<br\>  
              \&nbsp;\&nbsp;Linear Light Output: \<span class="tree-val" style="color:\#06b6d4;"\>\${res.dapc.percentLinear}\</span\>  
            \</div\>  
          \`;  
        }

        if (res.command) {  
          html \+= \`  
            \<div class="tree-node"\>  
              \<span class="tree-key"\>Standard Command Opcode:\</span\>\<br\>  
              \&nbsp;\&nbsp;Command: \<span class="tree-val" style="color:\#10b981; font-weight:700;"\>\${res.command.commandName} (\${res.command.opcode})\</span\>\<br\>  
              \&nbsp;\&nbsp;Classification: \<span class="tree-val"\>\${res.command.isConfigCommand ? 'Configuration / Memory Write' : (res.command.isQueryCommand ? 'Telemetry Query' : 'Direct Action')}\</span\>  
            \</div\>  
          \`;  
        }  
      }

      output.innerHTML \= html;

      // Manchester waveform  
      const mch \= engine.encodeManchester(bytes);  
      document.getElementById('manchester-output').innerText \= \`Bitstream: \${mch.bitStream} (16 data \+ 1 start)\\nManchester Waveform (32 half-bits):\\n\${mch.manchesterHalfBits}\\nDuration: \${mch.durationUs.toFixed(1)} µs at 1200 baud\`;

    } catch (e) {  
      output.innerHTML \= \`\<p style="color: var(--accent-rose); font-weight:600;"\>Dissection Error: \${e.message}\</p\>\`;  
    }  
  }

  // Initial load  
  runDissection();

  // DAPC Studio Target Type toggling  
  const targetType \= document.getElementById('dali-target-type');  
  const groupTargetVal \= document.getElementById('group-target-val');  
  const labelTargetVal \= document.getElementById('label-target-val');  
  targetType.addEventListener('change', () \=\> {  
    const val \= targetType.value;  
    if (val \=== 'short') {  
      groupTargetVal.style.display \= 'block';  
      labelTargetVal.innerText \= 'Short Address (0 to 63)';  
      document.getElementById('dali-target-val').max \= 63;  
    } else if (val \=== 'group') {  
      groupTargetVal.style.display \= 'block';  
      labelTargetVal.innerText \= 'Group Address (0 to 15)';  
      document.getElementById('dali-target-val').max \= 15;  
    } else {  
      groupTargetVal.style.display \= 'none';  
    }  
  });

  // Operation mode toggling (DAPC vs Command)  
  const daliMode \= document.getElementById('dali-mode');  
  const groupDapc \= document.getElementById('group-dapc-controls');  
  const groupCmd \= document.getElementById('group-cmd-controls');  
  daliMode.addEventListener('change', () \=\> {  
    const isDapc \= (daliMode.value \=== 'dapc');  
    groupDapc.style.display \= isDapc ? 'block' : 'none';  
    groupCmd.style.display \= isDapc ? 'none' : 'block';  
  });

  // Arc Power Slider sync  
  const arcSlider \= document.getElementById('dali-arc-slider');  
  const arcInput \= document.getElementById('dali-arc-input');

  function updateArcPercents(level) {  
    const logPct \= engine.dapcLevelToPercentLog(level);  
    const linPct \= engine.dapcLevelToPercentLinear(level);  
    document.getElementById('val-log-percent').innerText \= (level \=== 0\) ? '0.0% (OFF)' : \`\${logPct.toFixed(2)}%\`;  
    document.getElementById('val-lin-percent').innerText \= (level \=== 0\) ? '0.0% (OFF)' : \`\${linPct.toFixed(2)}%\`;  
  }

  arcSlider.addEventListener('input', () \=\> {  
    const val \= parseInt(arcSlider.value, 10);  
    arcInput.value \= val;  
    updateArcPercents(val);  
  });  
  arcInput.addEventListener('input', () \=\> {  
    const val \= parseInt(arcInput.value, 10\) \|\| 0;  
    arcSlider.value \= val;  
    updateArcPercents(val);  
  });  
  updateArcPercents(254);

  // Build DALI Frame  
  document.getElementById('btn-build-frame').addEventListener('click', () \=\> {  
    const tType \= targetType.value;  
    const tVal \= parseInt(document.getElementById('dali-target-val').value, 10\) \|\| 0;  
    const isDapc \= (daliMode.value \=== 'dapc');

    let res;  
    if (isDapc) {  
      const level \= parseInt(arcInput.value, 10\) \|\| 0;  
      res \= engine.buildDapcFrame(tType, tVal, level);  
      document.getElementById('synth-hex-output').innerText \= res.hexString;  
      document.getElementById('synth-breakdown').innerHTML \= \`  
        \<div\>\<strong\>Frame Type:\</strong\> Direct Arc Power Control (DAPC)\</div\>  
        \<div\>\<strong\>Raw Bytes:\</strong\> \<code\>0x\${res.hexString}\</code\>\</div\>  
        \<div\>\<strong\>Arc Level:\</strong\> \${res.level} / 254\</div\>  
        \<div\>\<strong\>Logarithmic Dimming Output:\</strong\> \${(res.level \=== 0\) ? '0%' : res.percentLog.toFixed(2) \+ '%'}\</div\>  
        \<div\>\<strong\>Linear Light Output:\</strong\> \${(res.level \=== 0\) ? '0%' : res.percentLinear.toFixed(2) \+ '%'}\</div\>  
      \`;  
    } else {  
      const cmdCode \= parseInt(document.getElementById('dali-cmd-select').value, 10);  
      res \= engine.buildCommandFrame(tType, tVal, cmdCode);  
      document.getElementById('synth-hex-output').innerText \= res.hexString;  
      document.getElementById('synth-breakdown').innerHTML \= \`  
        \<div\>\<strong\>Frame Type:\</strong\> Standard Command (IEC 62386-102)\</div\>  
        \<div\>\<strong\>Raw Bytes:\</strong\> \<code\>0x\${res.hexString}\</code\>\</div\>  
        \<div\>\<strong\>Command:\</strong\> \${res.commandName} (0x\${cmdCode.toString(16)})\</div\>  
      \`;  
    }  
  });

  // DT8 Color Temp Slider  
  const kelvinSlider \= document.getElementById('dt8-kelvin-slider');  
  const kelvinInput \= document.getElementById('dt8-kelvin-input');  
  const mirekVal \= document.getElementById('dt8-mirek-val');

  function updateKelvin(k) {  
    const m \= engine.kelvinToMirek(k);  
    mirekVal.innerText \= \`\${m} mirek\`;  
  }  
  kelvinSlider.addEventListener('input', () \=\> {  
    const val \= parseInt(kelvinSlider.value, 10);  
    kelvinInput.value \= val;  
    updateKelvin(val);  
  });  
  kelvinInput.addEventListener('input', () \=\> {  
    const val \= parseInt(kelvinInput.value, 10\) \|\| 4000;  
    kelvinSlider.value \= val;  
    updateKelvin(val);  
  });  
  updateKelvin(4000);

  document.getElementById('btn-build-dt8').addEventListener('click', () \=\> {  
    const k \= parseInt(kelvinInput.value, 10\) \|\| 4000;  
    const seq \= engine.buildDt8ColorTempSequence('broadcast', 0, k);  
    document.getElementById('dt8-seq-output').innerText \= seq.hexSequence;  
  });

  // D4i Energy Parser  
  document.getElementById('btn-parse-d4i').addEventListener('click', () \=\> {  
    const hex \= document.getElementById('d4i-hex-input').value.trim();  
    const output \= document.getElementById('d4i-output');  
    try {  
      const bytes \= engine.hexToBytes(hex);  
      const res \= engine.parseD4iEnergyBank(bytes);  
      if (\!res.valid) {  
        output.innerHTML \= \`\<p style="color:var(--accent-rose);"\>\${res.error}\</p\>\`;  
        return;  
      }  
      output.innerHTML \= \`  
        \<div\>\<strong\>Standard:\</strong\> \${res.standard}\</div\>  
        \<div\>\<strong\>Cumulative Active Energy:\</strong\> \<span style="color:var(--accent-dali); font-weight:700;"\>\${res.activeEnergyKWh} kWh\</span\> (\${res.activeEnergyWh} Wh)\</div\>  
        \<div\>\<strong\>Active Power:\</strong\> \<span style="color:\#06b6d4; font-weight:700;"\>\${res.activePowerWatts} W\</span\>\</div\>  
        \<div\>\<strong\>Power Factor:\</strong\> \${res.powerFactor}\</div\>  
      \`;  
    } catch (e) {  
      output.innerHTML \= \`\<p style="color:var(--accent-rose);"\>Error: \${e.message}\</p\>\`;  
    }  
  });

  // PCAP Export  
  document.getElementById('btn-export-pcap').addEventListener('click', () \=\> {  
    const hex \= hexInput.value.trim();  
    if (\!hex) return;  
    const bytes \= engine.hexToBytes(hex);  
    const pcapBytes \= engine.exportPcap(\[bytes\]);  
    downloadBlob(pcapBytes, 'dali-capture.pcap');  
  });

  function downloadBlob(bytes, filename) {  
    const blob \= new Blob(\[bytes\], { type: 'application/vnd.tcpdump.pcap' });  
    const url \= URL.createObjectURL(blob);  
    const a \= document.createElement('a');  
    a.href \= url;  
    a.download \= filename;  
    a.click();  
    URL.revokeObjectURL(url);  
  }

  // Security Auditor  
  function runAudit() {  
    const sampleConfig \= {  
      hasBroadcastProtection: false,  
      hasEmergencyTesting: true,  
      isZoned: false,  
      isCommissioningLocked: true,  
      hasD4iProtection: true,  
      hasStrobeProtection: false,  
      hasSensorCorrelation: true,  
      hasBusPowerMonitoring: true,  
      hasSignedFirmware: true,  
      hasEnclosureTamper: false  
    };

    const res \= auditor.auditConfiguration(sampleConfig);  
    document.getElementById('audit-score').innerText \= res.score;  
    document.getElementById('audit-posture').innerText \= \`\${res.posture} POSTURE\`;

    const tbody \= document.getElementById('audit-tbody');  
    tbody.innerHTML \= res.findings.map(f \=\> \`  
      \<tr\>  
        \<td\>\<code\>\${f.id}\</code\>\</td\>  
        \<td\>\<strong\>\${f.title}\</strong\>\</td\>  
        \<td\>\<small\>\${f.standard}\</small\>\</td\>  
        \<td\>\<span class="badge \${f.severity \=== 'CRITICAL' ? 'badge-rose' : f.severity \=== 'HIGH' ? 'badge-dali' : 'badge-cyan'}"\>\${f.severity}\</span\>\</td\>  
        \<td\>\<span class="badge \${f.status \=== 'PASS' ? 'badge-emerald' : 'badge-rose'}"\>\${f.status}\</span\>\</td\>  
        \<td\>\<small\>\${f.remediation}\</small\>\</td\>  
      \</tr\>  
    \`).join('');  
  }

  document.getElementById('btn-run-audit').addEventListener('click', runAudit);  
  runAudit();

  // Config Synthesizer  
  document.getElementById('nftables-output').innerText \= synthesizer.generateNftables();  
  document.getElementById('suricata-output').innerText \= synthesizer.generateSuricataRules();  
});  
