# SCADA System Penetration Testing Plan

> **IMPORTANT DISCLAIMER**
> This document is a **draft template for authorized security testing only**.
> Penetration testing must only be performed on systems you **own or have explicit written authorization** to test.
> Unauthorized testing of SCADA/ICS systems is illegal and may cause physical harm, equipment damage, or safety incidents.
> This plan must be reviewed and approved by your legal, compliance, and safety teams before execution.
> All findings are confidential — treat as Emerson Confidential.

---

## 1. Engagement Overview

| Field | Details |
|---|---|
| **Scope** | SCADA system as depicted in architecture diagram (scada.jpg) |
| **Test Type** | Black-box / Grey-box penetration test |
| **Methodology** | PTES, OWASP Testing Guide, IEC 62443, NIST SP 800-82 |
| **Authorization** | Written authorization required before commencement |
| **Environment** | Production (read-only observation) / Staging (active testing) |
| **Rules of Engagement** | See Section 3 |

---

## 2. Scope

### 2.1 In-Scope Assets (based on scada.jpg architecture)

| Layer | Component | Test Coverage |
|---|---|---|
| Control Center | HMI / Operator Workstation | Full |
| Control Center | Historian / Database Server | Full |
| Control Center | SCADA Server (MTU) | Full |
| Communication | Ethernet / Fiber / Radio / Cellular links | Network-layer |
| Communication | Protocols: Modbus, DNP3, IEC 60870-5 | Protocol analysis |
| Field Controllers | RTU 1, RTU 2 | Limited (see Rules) |
| Field Controllers | PLC 1, PLC 2 | Limited (see Rules) |
| Field Instruments | Sensors, Actuators, Valves, Motors, Pumps | Observation only |
| Physical Process | Water Treatment / Power / Pipeline / Manufacturing | Out of scope — safety risk |

### 2.2 Out-of-Scope
- Physical process equipment (valves, pumps, motors) — safety risk
- Live production PLCs/RTUs without explicit written approval and a maintenance window
- Third-party vendor systems not owned by the organization
- Denial-of-service attacks against live control systems

---

## 3. Rules of Engagement

1. **Written authorization** signed by system owner and safety officer before any testing begins.
2. **Safety first** — immediately halt testing if any unintended process impact is observed; contact the on-call safety engineer.
3. **No destructive commands** — no writes, downloads, or firmware changes to PLCs/RTUs without explicit approval and a test window.
4. **Maintenance window** — active field-device testing only during agreed downtime windows.
5. **Communication channel** — maintain a dedicated out-of-band comms channel (phone/radio) with the operations team throughout.
6. **Emergency stop procedure** — document and agree on a kill-switch procedure before testing starts.
7. **Data handling** — all findings, screenshots, and captures are Emerson Confidential; store encrypted, share on need-to-know basis.

---

## 4. Team & Roles

| Role | Responsibility |
|---|---|
| Lead Tester (ICS specialist) | Overall test execution, protocol-level testing |
| Network Tester | Network scanning, traffic analysis, pivoting |
| Application Tester | HMI, historian, SCADA server application testing |
| Safety Monitor (OT staff) | Real-time process observation; authority to halt testing |
| Scribe / Evidence Collector | Document all actions, timestamps, and findings |

---

## 5. Testing Phases

### Phase 1 — Reconnaissance & Intelligence Gathering

**Objective:** Build a detailed picture of the target environment without actively probing.

| Activity | Technique | Tools |
|---|---|---|
| Passive network traffic capture | Capture and analyze SCADA traffic | Wireshark, tcpdump |
| Architecture review | Review provided diagrams, configs | Manual review |
| Asset enumeration | Identify all IP addresses, device types | Network documentation |
| Protocol identification | Identify protocols in use (Modbus, DNP3, etc.) | Protocol analyzers |
| OSINT | Vendor manuals, firmware advisories, CVE databases | Shodan (external), NVD |
| Physical walkthrough | Identify exposed ports, labeling, access controls | Manual inspection |

**Deliverable:** Asset inventory, network map, protocol list.

---

### Phase 2 — Network Scanning & Enumeration

**Objective:** Map the network topology and identify open services.

| Activity | Technique | Caution |
|---|---|---|
| Host discovery | Slow, low-noise ping sweep | Use minimal packet rates — OT devices crash easily |
| Port/service scanning | TCP/UDP scan on key SCADA ports (502, 20000, 2404, 102) | Rate-limit aggressively |
| OS/device fingerprinting | Banner grabbing, TTL analysis | Avoid aggressive fingerprinting |
| VLAN/segmentation mapping | Identify trust boundary gaps | |
| Firewall / ACL rule review | Configuration review (if grey-box) | |
| Wireless survey | Check for unauthorized Wi-Fi / radio channels | SDR, WiFi analyzer |

**Key SCADA Ports:**

| Port | Protocol | Component |
|---|---|---|
| 502 | Modbus TCP | PLCs / RTUs |
| 20000 | DNP3 | RTUs |
| 2404 | IEC 60870-5-104 | RTUs |
| 102 | IEC 61850 / S7 | PLCs |
| 44818 | EtherNet/IP | PLCs |
| 4840 | OPC-UA | SCADA Server |

**Deliverable:** Confirmed network topology, open services map, segmentation gaps.

---

### Phase 3 — Vulnerability Assessment

**Objective:** Identify known vulnerabilities without exploiting them.

| Activity | Detail |
|---|---|
| CVE / patch level review | Check firmware/software versions against NVD/ICS-CERT advisories |
| Default credential check | Test known vendor default credentials on HMI, RTU, PLC interfaces |
| Configuration review | Review SCADA server, historian, and HMI configurations |
| Protocol weakness analysis | Identify unauthenticated Modbus/DNP3 function codes |
| Certificate / encryption review | Check for weak/missing TLS on historian, OPC-UA, remote access |
| Remote access audit | Identify VPN, RDP, cellular modem access points |
| Physical security review | Unlocked cabinets, USB ports, exposed serial consoles |

**Tools:** Nessus (ICS plugins), Claroty / Dragos / Tenable.OT, SCADAPy, ModbusPal.

**Deliverable:** Vulnerability list with CVE references, severity ratings (CVSS), and affected components.

---

### Phase 4 — Exploitation (Controlled / Authorized Only)

> Only proceed with explicit written approval; only in an agreed maintenance window; safety monitor must be present.

#### 4.1 Control Center (HMI / SCADA Server / Historian)

| Test Case | Technique | Expected Finding |
|---|---|---|
| Authentication bypass | Brute-force/credential stuffing on HMI login | Weak password policy |
| Session management | Hijack HMI session tokens | Insecure session handling |
| SQL injection | Test historian DB queries via HMI inputs | Data exfiltration |
| Privilege escalation | Exploit local OS vulnerabilities | Admin access |
| Lateral movement | Pivot from HMI to SCADA server | Flat network exposure |
| Malicious HMI script | Inject script/macro into HMI display | XSS / script execution |

#### 4.2 Communication Network / Protocols

| Test Case | Technique | Expected Finding |
|---|---|---|
| Modbus unauthenticated read | Read coils/registers via Modbus function codes | Exposed process data |
| Modbus unauthenticated write | Write to holding registers (test only — controlled) | Unauthorized command |
| DNP3 spoofing | Craft DNP3 packets impersonating the MTU | No source authentication |
| Man-in-the-middle | ARP spoofing on OT network segment | Cleartext traffic |
| Replay attack | Capture and replay valid control packets | Commands re-executed |
| Rogue device | Connect unapproved device to OT network | No NAC enforcement |

#### 4.3 Field Controllers (RTU / PLC) — Maintenance Window Only

| Test Case | Technique | Expected Finding |
|---|---|---|
| Default credentials | Test vendor defaults on engineering port | Unauthorized access |
| Unauthenticated read | Read PLC memory / I/O over native protocol | Process data exposed |
| Logic read | Download current ladder logic | IP theft, reconnaissance |
| Firmware version | Identify outdated firmware against CVE list | Known vulnerabilities |
| Debug port access | Access serial/JTAG/Ethernet debug interface | Unrestricted maintenance access |

**Tools:** Metasploit (ICS modules), SCADAPy, ModbusPal, DNP3 tools, PLCScan, OpenPLC.

**Deliverable:** Exploitation evidence (screenshots, packet captures), confirmed impact.

---

### Phase 5 — Post-Exploitation & Lateral Movement (Controlled)

**Objective:** Demonstrate the blast radius of a successful compromise.

| Activity | Detail |
|---|---|
| IT → OT pivot | From a compromised HMI, attempt to reach PLCs/RTUs |
| Persistence simulation | Demonstrate how an attacker could maintain access (do NOT actually install) |
| Data exfiltration simulation | Demonstrate access to historian data / process setpoints |
| Operator "blind" simulation | Demonstrate ability to falsify HMI readings |
| Safety system access | Identify if the safety system (SIS) is reachable from compromised OT network |

**Deliverable:** Attack path diagrams, documented pivot chains.

---

### Phase 6 — Reporting & Remediation Guidance

**Objective:** Document all findings clearly and provide actionable remediation steps.

#### Report Structure

1. Executive Summary — business risk, overall risk rating
2. Scope and methodology
3. Attack narrative — step-by-step story of the compromise chain
4. Findings register — one entry per finding:
   - Title, CVSS score, affected component
   - Description, evidence (screenshot/packet capture)
   - Business impact
   - Remediation recommendation
   - References (CVE, CWE, ICS-CERT)
5. Risk heat map
6. Remediation roadmap (short/medium/long term)
7. Appendices — raw tool output, raw evidence

#### Risk Rating Scale

| Rating | CVSS Range | Definition |
|---|---|---|
| Critical | 9.0–10.0 | Immediate risk to safety or process integrity |
| High | 7.0–8.9 | Significant risk; exploit path confirmed |
| Medium | 4.0–6.9 | Risk present but requires additional conditions |
| Low | 0.1–3.9 | Minimal risk; defense-in-depth gap |
| Informational | N/A | Best practice / configuration improvement |

---

## 6. Tools & References

### Recommended Toolset

| Category | Tools |
|---|---|
| Network scanning | Nmap (slow/careful), Wireshark, tcpdump |
| ICS/OT scanning | Claroty, Dragos, Tenable.OT, Nessus ICS plugins |
| Protocol testing | SCADAPy, ModbusPal, PyModbus, DNP3 tools |
| Exploitation | Metasploit ICS modules, custom scripts |
| Wireless | Aircrack-ng, SDR (HackRF) for radio links |
| Reporting | Dradis, Serpico, or Word/Excel |

### Standards & References

| Standard | Relevance |
|---|---|
| IEC 62443 | Industrial cybersecurity standard — primary reference |
| NIST SP 800-82 | Guide to ICS Security |
| NIST SP 800-115 | Technical Guide to Pen Testing |
| MITRE ATT&CK for ICS | Tactics, techniques, and procedures |
| ICS-CERT Advisories | Vendor-specific CVEs and mitigations |
| PTES | Penetration Testing Execution Standard |

---

## 7. Safety & Emergency Procedures

| Trigger | Action |
|---|---|
| Any unintended process change | **STOP ALL TESTING IMMEDIATELY** — call safety engineer on hotline |
| Loss of communication with operations team | Halt testing; re-establish contact before resuming |
| Controller unresponsive after test | Escalate to OT engineer; do not attempt recovery without authorization |
| Physical alarm triggered | Halt testing; notify control room operator immediately |

**Emergency contact (fill in before testing begins):**
- Safety Engineer: _______________
- OT Operations Lead: _______________
- Incident Response: _______________

---

## 8. Timeline (Sample)

| Week | Activity |
|---|---|
| Week 1 | Kick-off, documentation review, passive reconnaissance |
| Week 2 | Network scanning, vulnerability assessment |
| Week 3 | Exploitation (maintenance window scheduled) |
| Week 4 | Post-exploitation, lateral movement testing |
| Week 5 | Report writing and draft delivery |
| Week 6 | Debrief, remediation planning, final report |

---

*This document is an unsubstantiated draft. It must be reviewed by your security, legal, and safety teams and approved before any testing activity commences. Legal review is required before this plan enters any customer-facing or contractual context.*
