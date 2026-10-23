/\*\*  
 \* DALIForge Pro \- IEC 62386 Smart Lighting Cybersecurity & Life-Safety Auditor  
 \* Standards: IEC 62386-101/102/202, EN 50172 (Emergency Lighting), NIST SP 800-82  
 \* Author: Autonomous Digital Product Factory (Product \#171)  
 \* Version: 1.0.0  
 \*/

class DALISecurityAuditor {  
  constructor() {  
    this.RULES \= \[  
      {  
        id: 'DALI-SEC-01',  
        title: 'Unrestricted Broadcast Blackout Command Exposure (DAPC 0x00 / OFF)',  
        standard: 'Life Safety Code NFPA 101 / IEC 62386-102 Section 9.2',  
        severity: 'CRITICAL',  
        weight: 20,  
        description: 'Unauthenticated DALI broadcast commands (0xFE 0x00 / 0xFF 0x00) over IP/BMS gateways can instantly plunge entire commercial buildings into darkness.',  
        remediation: 'Implement gateway rate-limiting and require multi-factor authorization before forwarding broadcast Turn-Off frames to physical DALI loops.'  
      },  
      {  
        id: 'DALI-SEC-02',  
        title: 'Emergency Lighting Automated Functional Testing (IEC 62386-202)',  
        standard: 'EN 50172 / BS 5266-1 / IEC 62386-202 (Emergency Control Gear)',  
        severity: 'CRITICAL',  
        weight: 20,  
        description: 'Self-contained emergency luminaires failing to perform automated monthly functional and annual duration tests leave escape paths unlit during fires.',  
        remediation: 'Automate IEC 62386-202 Function 240 (Start Function Test) scheduling and continuously audit battery status in central BMS.'  
      },  
      {  
        id: 'DALI-SEC-03',  
        title: 'DALI-to-IP / BACnet Gateway Network Segmentation',  
        standard: 'IEC 62443-3-2 (Zoning & Conduit Specification) / NIST SP 800-82',  
        severity: 'HIGH',  
        weight: 15,  
        description: 'Directly routing DALI gateway commissioning interfaces (HTTP/BACnet/KNX) on enterprise corporate LANs exposes lighting to lateral malware propagation.',  
        remediation: 'Isolate DALI gateways on dedicated building automation VLANs (Purdue Level 1/2) with stateful firewall access controls.'  
      },  
      {  
        id: 'DALI-SEC-04',  
        title: 'Runtime Short Address Re-Commissioning Lockout',  
        standard: 'IEC 62386-102 Section 9.5 (Commissioning & Addressing)',  
        severity: 'HIGH',  
        weight: 15,  
        description: 'Allowing special commissioning commands (INITIALIZE 0xA1, RANDOMISE 0xA3) during normal operating hours allows rogue nodes to remap light fixtures.',  
        remediation: 'Disable DALI commissioning mode at the gateway during normal operations; enforce maintenance-window authorization for address allocation.'  
      },  
      {  
        id: 'DALI-SEC-05',  
        title: 'D4i Part 252 Energy Telemetry Integrity & Tamper Protection',  
        standard: 'D4i Specification Part 252 (Energy Reporting) / ISO 50001',  
        severity: 'MEDIUM',  
        weight: 10,  
        description: 'Unprotected Memory Bank 202 access allows manipulation of energy consumption logs (kWh) and power factor readings used in green building certifications.',  
        remediation: 'Enforce write-protection on Memory Bank 202 and verify cryptographic checksums on automated energy billing exports.'  
      },  
      {  
        id: 'DALI-SEC-06',  
        title: 'Circadian Tunable White Stroboscopic / Flashing Hazard Defense',  
        standard: 'IEEE 1789-2015 (Health Risks from Light Flashing / Photoepilepsy)',  
        severity: 'MEDIUM',  
        weight: 5,  
        description: 'Rapid, unauthorized oscillation of DT8 color temperature or DAPC levels can trigger photoepileptic seizures and visual disorientation.',  
        remediation: 'Enforce minimum fade times (Fade Time register \> 0.7s) and block rapid high-frequency arc power strobing on gateway interfaces.'  
      },  
      {  
        id: 'DALI-SEC-07',  
        title: 'Occupancy & Daylight Sensor (IEC 62386-103) Event Spoofing',  
        standard: 'IEC 62386-103 (Control Devices) / ASHRAE 90.1',  
        severity: 'MEDIUM',  
        weight: 5,  
        description: 'Rogue 24-bit input device frames simulating continuous human occupancy defeat energy-saving vacancy timeouts.',  
        remediation: 'Cross-correlate DALI occupancy telemetry with BACnet HVAC presence sensors to detect anomalous presence assertions.'  
      },  
      {  
        id: 'DALI-SEC-08',  
        title: 'DALI Bus Power Supply Voltage & Current Overload Monitoring',  
        standard: 'IEC 62386-101 Section 4 (Electrical Specifications)',  
        severity: 'LOW',  
        weight: 4,  
        description: 'DALI bus voltage sagging below 11.5V or current exceeding 250mA causes frame corruption and communication failures across all 64 gear.',  
        remediation: 'Install intelligent DALI power supplies with voltage/current telemetry reporting to the central facility SCADA.'  
      },  
      {  
        id: 'DALI-SEC-09',  
        title: 'Gateway Firmware Integrity & Cryptographic Signing',  
        standard: 'NIST SP 800-193 (Platform Firmware Resiliency)',  
        severity: 'LOW',  
        weight: 3,  
        description: 'Unsigned gateway firmware updates allow installation of modified images that siphon building occupancy data.',  
        remediation: 'Enforce secure boot and cryptographic RSA/ECDSA signature verification on all DALI gateway firmware images.'  
      },  
      {  
        id: 'DALI-SEC-10',  
        title: 'Physical Lighting Riser & Enclosure Tamper Auditing',  
        standard: 'IEC 62443-4-2 CR 1.7 (Physical Tamper Protection)',  
        severity: 'LOW',  
        weight: 3,  
        description: 'Unmonitored electrical riser cabinets allow unauthorized tap-ins to the two-wire DALI polarity-free bus.',  
        remediation: 'Fit electrical riser enclosures with supervised door contact switches reporting to the building access control system.'  
      }  
    \];  
  }

  auditConfiguration(config \= {}) {  
    const findings \= \[\];  
    let deduction \= 0;

    // 1\. Broadcast Blackout Protection  
    if (\!config.hasBroadcastProtection) {  
      findings.push({ ...this.RULES\[0\], status: 'FAIL', current: 'Unfiltered Broadcast OFF (0xFE 0x00) Forwarding Allowed' });  
      deduction \+= this.RULES\[0\].weight;  
    } else {  
      findings.push({ ...this.RULES\[0\], status: 'PASS', current: 'Gateway Broadcast Rate-Limiting & Approval Enforced' });  
    }

    // 2\. Emergency Lighting Automated Testing  
    if (\!config.hasEmergencyTesting) {  
      findings.push({ ...this.RULES\[1\], status: 'FAIL', current: 'Manual or Missing IEC 62386-202 Emergency Battery Tests' });  
      deduction \+= this.RULES\[1\].weight;  
    } else {  
      findings.push({ ...this.RULES\[1\], status: 'PASS', current: 'Automated Monthly Functional & Annual Duration Tests Active' });  
    }

    // 3\. Network Segmentation  
    if (\!config.isZoned) {  
      findings.push({ ...this.RULES\[2\], status: 'FAIL', current: 'DALI Gateway Exposed Directly on Corporate LAN' });  
      deduction \+= this.RULES\[2\].weight;  
    } else {  
      findings.push({ ...this.RULES\[2\], status: 'PASS', current: 'Dedicated Building Automation VLAN & Firewall Conduit Active' });  
    }

    // 4\. Commissioning Lockout  
    if (\!config.isCommissioningLocked) {  
      findings.push({ ...this.RULES\[3\], status: 'FAIL', current: 'INITIALIZE (0xA1) & RANDOMISE Permitted at Runtime' });  
      deduction \+= this.RULES\[3\].weight;  
    } else {  
      findings.push({ ...this.RULES\[3\], status: 'PASS', current: 'Runtime Commissioning Lockout Active' });  
    }

    // 5\. D4i Energy Integrity  
    if (\!config.hasD4iProtection) {  
      findings.push({ ...this.RULES\[4\], status: 'FAIL', current: 'Memory Bank 202 Energy Data Unprotected' });  
      deduction \+= this.RULES\[4\].weight;  
    } else {  
      findings.push({ ...this.RULES\[4\], status: 'PASS', current: 'D4i Energy Bank Write-Protect & Telemetry Auditing Enforced' });  
    }

    // 6\. Circadian Stroboscopic Protection  
    if (\!config.hasStrobeProtection) {  
      findings.push({ ...this.RULES\[5\], status: 'FAIL', current: 'Instant High-Frequency Arc Power Cycling Permitted' });  
      deduction \+= this.RULES\[5\].weight;  
    } else {  
      findings.push({ ...this.RULES\[5\], status: 'PASS', current: 'Minimum Fade Time & Anti-Stroboscopic Filtering Enforced' });  
    }

    // 7\. Sensor Spoofing  
    if (\!config.hasSensorCorrelation) {  
      findings.push({ ...this.RULES\[6\], status: 'FAIL', current: 'Uncorrelated DALI-2 Input Device Events Accepted' });  
      deduction \+= this.RULES\[6\].weight;  
    } else {  
      findings.push({ ...this.RULES\[6\], status: 'PASS', current: 'Multi-Sensor Occupancy Cross-Correlation Active' });  
    }

    // 8\. Power Supply Telemetry  
    if (\!config.hasBusPowerMonitoring) {  
      findings.push({ ...this.RULES\[7\], status: 'FAIL', current: 'No Real-Time DALI Bus Voltage/Current Telemetry' });  
      deduction \+= this.RULES\[7\].weight;  
    } else {  
      findings.push({ ...this.RULES\[7\], status: 'PASS', current: '16V DALI Bus Power & 250mA Clamp Supervised' });  
    }

    // 9\. Firmware Signing  
    if (\!config.hasSignedFirmware) {  
      findings.push({ ...this.RULES\[8\], status: 'FAIL', current: 'Unsigned Gateway Firmware Updates Allowed' });  
      deduction \+= this.RULES\[8\].weight;  
    } else {  
      findings.push({ ...this.RULES\[8\], status: 'PASS', current: 'Cryptographic RSA/ECDSA Firmware Verification Active' });  
    }

    // 10\. Physical Tampering  
    if (\!config.hasEnclosureTamper) {  
      findings.push({ ...this.RULES\[9\], status: 'FAIL', current: 'Lighting Riser Cabinets Unsupervised' });  
      deduction \+= this.RULES\[9\].weight;  
    } else {  
      findings.push({ ...this.RULES\[9\], status: 'PASS', current: 'Monitored Door Contacts on Lighting Panels' });  
    }

    const score \= Math.max(0, 100 \- deduction);  
    let posture \= 'POOR';  
    if (score \>= 90\) posture \= 'EXCELLENT';  
    else if (score \>= 75\) posture \= 'GOOD';  
    else if (score \>= 50\) posture \= 'MODERATE';

    return {  
      score,  
      posture,  
      findings,  
      timestamp: new Date().toISOString()  
    };  
  }  
}

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= { DALISecurityAuditor };  
}  
