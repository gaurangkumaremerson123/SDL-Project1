# SCADA Penetration Test — Test Execution Matrix

> **Companion to:** `SCADA-Penetration-Testing-Plan.md`
> **Classification:** Emerson Confidential — share on need-to-know basis
> **Status legend:** ⬜ Not Started · 🟦 In Progress · ✅ Pass (no issue) · ❌ Fail (finding) · ⏭️ Skipped · 🚫 Blocked
> **Authorization required before any active test. Safety monitor must be present for OT-affecting tests.**

---

## How to Use This Matrix

Each test case has a unique ID (`TC-<phase>-<seq>`). During execution, fill in **Status**, **Tester**, **Date**, **Evidence Ref**, and **Finding ID** (link to the report's findings register). Use the **Risk** column to pre-prioritize and the **OT Impact** column to flag tests that require a maintenance window or safety sign-off.

| OT Impact flag | Meaning |
|---|---|
| 🟢 Safe | Passive / read-only; low risk to operations |
| 🟡 Caution | Active probing; rate-limit, monitor process |
| 🔴 Restricted | May affect control/process; **maintenance window + written approval + safety monitor required** |

---

## Phase 1 — Reconnaissance & Intelligence Gathering

| TC ID | Test Case | Target | Technique / Tool | OT Impact | Risk | Status | Tester | Date | Evidence Ref | Finding ID |
|---|---|---|---|---|---|---|---|---|---|---|
| TC-1-01 | Passive network traffic capture | OT network segment | Wireshark / tcpdump | 🟢 Safe | Info | ⬜ | | | | |
| TC-1-02 | Architecture & document review | Provided diagrams/configs | Manual review | 🟢 Safe | Info | ⬜ | | | | |
| TC-1-03 | Asset enumeration (IPs/device types) | All in-scope hosts | Network docs / passive | 🟢 Safe | Info | ⬜ | | | | |
| TC-1-04 | Protocol identification | Comms network | Protocol analyzer | 🟢 Safe | Info | ⬜ | | | | |
| TC-1-05 | OSINT — firmware/CVE advisories | Vendor/devices | NVD, ICS-CERT, Shodan (ext) | 🟢 Safe | Info | ⬜ | | | | |
| TC-1-06 | Physical walkthrough | Control room / cabinets | Manual inspection | 🟢 Safe | Low | ⬜ | | | | |

## Phase 2 — Network Scanning & Enumeration

| TC ID | Test Case | Target | Technique / Tool | OT Impact | Risk | Status | Tester | Date | Evidence Ref | Finding ID |
|---|---|---|---|---|---|---|---|---|---|---|
| TC-2-01 | Host discovery (low-noise sweep) | OT subnets | Nmap (rate-limited) | 🟡 Caution | Low | ⬜ | | | | |
| TC-2-02 | TCP/UDP port scan — SCADA ports | All hosts | Nmap (502/20000/2404/102/44818/4840) | 🟡 Caution | Medium | ⬜ | | | | |
| TC-2-03 | OS / device fingerprinting | Discovered hosts | Banner grab / TTL | 🟡 Caution | Low | ⬜ | | | | |
| TC-2-04 | VLAN / segmentation mapping | Network fabric | Topology analysis | 🟢 Safe | Medium | ⬜ | | | | |
| TC-2-05 | Firewall / ACL rule review | Boundary devices | Config review (grey-box) | 🟢 Safe | Medium | ⬜ | | | | |
| TC-2-06 | Wireless / radio survey | RF environment | SDR / WiFi analyzer | 🟡 Caution | Medium | ⬜ | | | | |

## Phase 3 — Vulnerability Assessment

| TC ID | Test Case | Target | Technique / Tool | OT Impact | Risk | Status | Tester | Date | Evidence Ref | Finding ID |
|---|---|---|---|---|---|---|---|---|---|---|
| TC-3-01 | CVE / patch level review | HMI, MTU, PLC, RTU, Historian | Version vs NVD/ICS-CERT | 🟢 Safe | High | ⬜ | | | | |
| TC-3-02 | Default credential check | HMI / RTU / PLC interfaces | Vendor default list | 🟡 Caution | High | ⬜ | | | | |
| TC-3-03 | Configuration review | SCADA server / HMI / historian | Manual / benchmark | 🟢 Safe | Medium | ⬜ | | | | |
| TC-3-04 | Protocol weakness analysis | Modbus / DNP3 / IEC 60870-5 | Function-code analysis | 🟡 Caution | High | ⬜ | | | | |
| TC-3-05 | TLS / certificate review | Historian, OPC-UA, remote access | sslyze / manual | 🟢 Safe | Medium | ⬜ | | | | |
| TC-3-06 | Remote access audit | VPN / RDP / cellular modem | Inventory + config review | 🟢 Safe | High | ⬜ | | | | |
| TC-3-07 | Physical security review | Cabinets / USB / serial consoles | Manual inspection | 🟢 Safe | Medium | ⬜ | | | | |

## Phase 4 — Exploitation (Authorized / Controlled Only)

### 4.1 Control Center (HMI / SCADA Server / Historian)

| TC ID | Test Case | Target | Technique / Tool | OT Impact | Risk | Status | Tester | Date | Evidence Ref | Finding ID |
|---|---|---|---|---|---|---|---|---|---|---|
| TC-4-01 | Authentication bypass / brute force | HMI login | Hydra / manual | 🟡 Caution | High | ⬜ | | | | |
| TC-4-02 | Session management / hijack | HMI sessions | Token analysis | 🟡 Caution | High | ⬜ | | | | |
| TC-4-03 | SQL injection | Historian DB via HMI inputs | sqlmap / manual | 🟡 Caution | High | ⬜ | | | | |
| TC-4-04 | Privilege escalation | HMI / server OS | Local exploit | 🔴 Restricted | High | ⬜ | | | | |
| TC-4-05 | Lateral movement | HMI → SCADA server | Pivot techniques | 🔴 Restricted | High | ⬜ | | | | |
| TC-4-06 | Malicious HMI script / display injection | HMI display | Script/macro injection | 🔴 Restricted | High | ⬜ | | | | |

### 4.2 Communication Network / Protocols

| TC ID | Test Case | Target | Technique / Tool | OT Impact | Risk | Status | Tester | Date | Evidence Ref | Finding ID |
|---|---|---|---|---|---|---|---|---|---|---|
| TC-4-07 | Modbus unauthenticated read | PLC/RTU | PyModbus / ModbusPal | 🟡 Caution | High | ⬜ | | | | |
| TC-4-08 | Modbus write (controlled) | Test PLC only | PyModbus | 🔴 Restricted | Critical | ⬜ | | | | |
| TC-4-09 | DNP3 spoofing (impersonate MTU) | RTU | DNP3 tools | 🔴 Restricted | Critical | ⬜ | | | | |
| TC-4-10 | Man-in-the-middle (ARP spoof) | OT segment | Ettercap / Bettercap | 🔴 Restricted | High | ⬜ | | | | |
| TC-4-11 | Replay attack | Control packets | Captured packet replay | 🔴 Restricted | Critical | ⬜ | | | | |
| TC-4-12 | Rogue device connection | OT switch port | Unapproved host + NAC test | 🟡 Caution | High | ⬜ | | | | |

### 4.3 Field Controllers (RTU / PLC) — Maintenance Window Only

| TC ID | Test Case | Target | Technique / Tool | OT Impact | Risk | Status | Tester | Date | Evidence Ref | Finding ID |
|---|---|---|---|---|---|---|---|---|---|---|
| TC-4-13 | Default credentials on eng. port | PLC1/PLC2/RTU1/RTU2 | Vendor defaults | 🔴 Restricted | High | ⬜ | | | | |
| TC-4-14 | Unauthenticated memory / I/O read | PLC / RTU | Native protocol read | 🔴 Restricted | High | ⬜ | | | | |
| TC-4-15 | Ladder logic download (read) | PLC | Engineering software | 🔴 Restricted | High | ⬜ | | | | |
| TC-4-16 | Firmware version vs CVE | PLC / RTU | Version check | 🟡 Caution | Medium | ⬜ | | | | |
| TC-4-17 | Debug / serial / JTAG port access | Controllers | Physical interface | 🔴 Restricted | High | ⬜ | | | | |

## Phase 5 — Post-Exploitation & Lateral Movement (Controlled)

| TC ID | Test Case | Target | Technique / Tool | OT Impact | Risk | Status | Tester | Date | Evidence Ref | Finding ID |
|---|---|---|---|---|---|---|---|---|---|---|
| TC-5-01 | IT → OT pivot | Compromised HMI → PLC/RTU | Pivot chain | 🔴 Restricted | Critical | ⬜ | | | | |
| TC-5-02 | Persistence simulation (no install) | Compromised host | Demonstrate only | 🔴 Restricted | High | ⬜ | | | | |
| TC-5-03 | Data exfiltration simulation | Historian / setpoints | Controlled extraction | 🟡 Caution | High | ⬜ | | | | |
| TC-5-04 | Operator "blind" / falsify HMI readings | HMI | Spoofed telemetry demo | 🔴 Restricted | Critical | ⬜ | | | | |
| TC-5-05 | Safety system (SIS) reachability | SIS from OT net | Connectivity test | 🔴 Restricted | Critical | ⬜ | | | | |

## Phase 6 — Reporting & Remediation

| TC ID | Activity | Deliverable | OT Impact | Status | Owner | Date | Evidence Ref |
|---|---|---|---|---|---|---|---|
| TC-6-01 | Consolidate findings register | CVSS-rated findings | 🟢 Safe | ⬜ | | | |
| TC-6-02 | Draft attack narrative | Step-by-step compromise story | 🟢 Safe | ⬜ | | | |
| TC-6-03 | Build risk heat map | Risk matrix | 🟢 Safe | ⬜ | | | |
| TC-6-04 | Remediation roadmap | Short/med/long-term plan | 🟢 Safe | ⬜ | | | |
| TC-6-05 | Executive summary | Business-level summary | 🟢 Safe | ⬜ | | | |
| TC-6-06 | Debrief & sign-off | Final report delivery | 🟢 Safe | ⬜ | | | |

---

## Execution Summary (fill in at close)

| Metric | Count |
|---|---|
| Total test cases | 39 |
| Passed (no finding) | |
| Failed (finding raised) | |
| Skipped | |
| Blocked | |
| Critical findings | |
| High findings | |
| Medium findings | |
| Low / Info findings | |

---

## Pre-Execution Checklist

- [ ] Signed written authorization obtained
- [ ] Rules of Engagement agreed and signed
- [ ] Maintenance window scheduled for 🔴 Restricted tests
- [ ] Safety monitor (OT staff) assigned and present
- [ ] Out-of-band emergency comms channel established
- [ ] Emergency stop / rollback procedure documented
- [ ] Backups of PLC/RTU configs taken
- [ ] Legal / compliance / safety review completed

---

*This document is an unsubstantiated draft template. Review with your security, legal, and safety teams and obtain written authorization before any active testing. Findings and evidence are Emerson Confidential.*
