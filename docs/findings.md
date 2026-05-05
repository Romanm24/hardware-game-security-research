# Research Findings

## Observations, Detection Challenges, and Defensive Recommendations

**Author:** Roman Mares — SOC Analyst in Training | U.S. Army Veteran  
**Research Type:** Defensive Cybersecurity / Endpoint Security  
**Environment:** Isolated lab (no live servers, no production systems)

---

## Overview

This document presents observations gathered during controlled lab research into hardware-based security threats in gaming environments. Findings are organized around three dimensions relevant to SOC operations: what was observed, what makes detection difficult, and what defenders can do.

All observations were made in an isolated test environment. No findings represent active exploitation of any live system, game server, or third-party platform.

---

## Section 1: Key Observations

### Observation 1 — Operating System Transparency to Hardware Device Identity

**Finding:** Host operating systems register hardware devices based entirely on self-reported device identifiers. No cryptographic challenge-response or hardware attestation mechanism exists in standard USB enumeration. The OS characterizes a device as whatever that device claims to be.

**Implication:** Any security policy, driver selection decision, or application-level trust determination that relies on USB device identity is operating on an unverified trust assumption.

**SOC Relevance:** Device allowlisting policies and USB control solutions are more brittle than they may appear. A device control policy that permits "known keyboards" provides meaningful protection against accidental or unsophisticated USB threats but provides limited protection against hardware capable of spoofing device identity.

---

### Observation 2 — Software Input Validation Operates Above the Hardware Layer

**Finding:** Input validation heuristics implemented in applications and kernel-level security drivers observe input events as delivered by the OS after processing through the USB HID stack. They do not have visibility into the physical origin of those input events.

**Implication:** Heuristic analysis designed to distinguish human input from automated input cannot reliably detect hardware-level input emulation, because from the software's perspective, hardware-emulated input is indistinguishable from human input.

**SOC Relevance:** Behavioral detections for input anomalies are valuable controls against software-based automation threats. They should not be assumed to cover hardware-based input threats. This gap should be documented in threat models and detection coverage matrices.

---

### Observation 3 — PCIe Hardware Access Operates Outside OS Memory Isolation

**Finding:** PCIe-connected devices interact with system memory through hardware pathways that are governed by hardware mechanisms (IOMMU/DMA remapping), not by OS process isolation. Systems where IOMMU is not properly enabled present a structural gap where hardware-level memory access is not bounded by OS security policy.

**Implication:** Security tools that rely on OS process isolation as their primary protection mechanism have a hardware-layer blind spot. This is a widely acknowledged limitation in endpoint security architecture, not a novel finding — but it is insufficiently understood in consumer and gaming endpoint contexts.

**SOC Relevance:** EDR solutions vary in their ability to detect hardware-level memory access anomalies. Understanding this limitation is important when assessing detection coverage and communicating risk to security stakeholders.

> See `technical-breakdown.md` sections 1.4 (IOMMU auditing), 1.5 (EDR visibility limits), and 1.7 (MITRE ATT&CK mapping) for detailed analysis.

---

### Observation 4 — Hardware Artifacts Are Subtle and Easily Overlooked

**Finding:** The introduction of DMA or input emulation hardware to a test system produced minimal immediately obvious system artifacts. The most observable signals were:

- New device entries in system device manager / USB device enumeration logs
- Unexpected driver loads corresponding to unrecognized hardware
- In some configurations, anomalous PCIe device entries not corresponding to the system's known hardware inventory
- Brief system pauses during device initialization that would be easily attributed to other causes

**Implication:** Hardware-based compromise is unlikely to generate high-confidence, high-fidelity alerts from standard software monitoring tools. Detection is most achievable through proactive hardware inventory management and physical security controls rather than reactive alert-based detection.

**SOC Relevance:** Low-and-slow hardware attacks may not produce SIEM alerts. This category of threat reinforces the value of asset management, hardware baseline documentation, and physical security integration in an enterprise security program.

---

### Observation 5 — The Threat Model Extends Beyond Gaming

**Finding:** The hardware-level attack surface documented in gaming contexts is conceptually identical to attack surfaces documented in enterprise, critical infrastructure, and government security research. The gaming context is notable for the accessibility of consumer-grade hardware that implements these capabilities, which lowers the barrier to entry compared to historically expensive or specialized hardware attack tools.

**Implication:** Security professionals who understand this threat landscape in the gaming context are observing a microcosm of broader hardware security challenges. The principles transfer directly to enterprise endpoint security, industrial control system security, and physical access threat modeling.

**SOC Relevance:** Awareness of hardware-based attack techniques prepares analysts to recognize analogous threats in higher-stakes environments where the same principles apply.

---

### Observation 6 — IOMMU Is Frequently Disabled or Unverified on Consumer Endpoints

**Finding:** IOMMU — the primary hardware-level control against unauthorized PCIe memory access — is not enabled by default on all consumer and workstation motherboards. Verifying IOMMU status requires explicit BIOS/UEFI inspection. OS-level indicators do not always surface IOMMU status clearly to end users or standard monitoring tools.

**Implication:** A significant proportion of consumer gaming endpoints may be running without active IOMMU enforcement, leaving them structurally exposed to PCIe-based memory access threats with no software-observable compensating control.

**SOC Relevance:** IOMMU status should be a standard line item in endpoint hardening audits. Detection programs that do not account for IOMMU status are operating with an incomplete picture of endpoint hardware risk posture.

> See `technical-breakdown.md` Section 1.4 for IOMMU audit procedures.

---

### Observation 7 — EDR Telemetry Has a Defined Hardware Observability Boundary

**Finding:** EDR-observable events associated with hardware introduction were limited to the driver installation and device enumeration phase. No EDR-observable events were generated by subsequent hardware-layer activity on systems without IOMMU enforcement.

**Implication:** The absence of EDR alerts does not confirm the absence of hardware-level activity. This detection gap must be explicitly documented in threat models and communicated clearly to stakeholders who may assume EDR provides comprehensive endpoint visibility.

**SOC Relevance:** Detection coverage matrices should explicitly note that hardware-layer activity below the OS observability boundary is outside standard EDR telemetry scope. Physical security, IOMMU enforcement, and hardware asset monitoring are the relevant compensating controls for this gap.

> See `technical-breakdown.md` Section 1.5 for a full breakdown of EDR visibility boundaries.

---

### Observation 8 — Physical Access Events Are Rarely Integrated Into SOC Workflows

**Finding:** Chassis intrusion detection and physical access control systems are widely available on enterprise-grade hardware and facilities infrastructure. However, these event streams are rarely integrated into SIEM platforms or SOC monitoring workflows. In most standard configurations, a chassis intrusion event would generate a local hardware alert with no corresponding SIEM entry, no analyst notification, and no incident response trigger.

**Implication:** The physical access precondition for PCIe-based hardware attacks is effectively unmonitored from a SOC perspective, even in organizations with otherwise mature detection programs.

**SOC Relevance:** Integrating chassis intrusion sensor events and physical access control logs into SIEM platforms — and correlating them with subsequent endpoint anomalies — is a high-value, relatively low-effort improvement to hardware threat detection coverage.

> See `technical-breakdown.md` Section 1.6 for the full physical security controls reference table.

---

## Section 2: Detection Challenges

### Challenge 1 — No Software-Observable Signal for Hardware Memory Access

Hardware DMA operations, in the absence of IOMMU enforcement, do not generate events in OS security logs, EDR telemetry, or SIEM data streams. The memory access occurs below the observability layer of software monitoring tools.

**Mitigation path:** IOMMU enforcement provides hardware-level visibility and containment. Physical access controls prevent the threat from being introduced. Hardware asset inventory detects the addition of unexpected PCIe devices.

---

### Challenge 2 — USB Device Identity Cannot Be Software-Verified

No standard OS mechanism allows software to cryptographically verify that a USB device's self-reported identity is accurate. Software-based device control policies that rely on VID/PID matching are operating on unverified data.

**Mitigation path:** Physical USB port controls, behavioral monitoring of HID device activity, and endpoint management solutions that enforce device class restrictions reduce (but do not eliminate) this risk.

---

### Challenge 3 — Input Emulation Is Behaviorally Indistinguishable at the Software Layer

Software-based input analysis cannot distinguish hardware-synthesized HID events from human-originated input events. Both arrive through identical OS pathways and carry identical metadata.

**Mitigation path:** Detection must focus on observable hardware-layer signals (device enumeration events, USB topology changes, unexpected HID device registrations) rather than input behavioral analysis alone.

---

### Challenge 4 — Consumer Hardware Availability Lowers Attacker Barrier

Hardware implementing these capabilities is commercially available, relatively inexpensive, and does not require specialized technical knowledge to acquire. This distinguishes the threat from historically expensive hardware attack tools associated with nation-state actors.

**Mitigation path:** Awareness of the threat category, physical security controls, and hardware asset management are proportionate and cost-effective countermeasures relative to the attacker's investment.

---

### Challenge 5 — Physical Access Is a Security Domain Often Siloed from SOC Operations

Many SOC teams operate primarily on network and endpoint telemetry. Physical security monitoring — chassis intrusion events, badge access logs, CCTV — is often managed by a separate function with limited integration into cybersecurity monitoring workflows.

**Mitigation path:** Integrate physical security event data into SIEM platforms where high-value endpoints are involved. Establish cross-functional communication between physical security and SOC teams. Include physical access anomalies in incident response playbooks.

---

## Section 3: Defensive Recommendations

### Recommendation 1 — Enable and Audit IOMMU on All Sensitive Endpoints

IOMMU (Intel VT-d / AMD-Vi) is the primary hardware-level defense against unauthorized DMA access. Verify that it is enabled in BIOS/UEFI settings and enforced at the OS level on endpoints where hardware-level memory protection is a concern.

**Priority:** High  
**Effort:** Low to Medium (configuration change; may require firmware and OS-level verification)  
**Detection improvement:** Significant — converts a silent hardware threat into a potentially containable one

---

### Recommendation 2 — Implement Hardware Asset Inventory with Baseline Monitoring

Maintain a documented baseline of all hardware components — including PCIe devices and USB devices — for sensitive endpoints. Monitor for deviations from this baseline using endpoint management tools, EDR solutions, or OS-level device enumeration logs.

**Priority:** High  
**Effort:** Medium (requires initial baselining and ongoing monitoring)  
**Detection improvement:** Moderate — provides early warning for hardware introduction events

---

### Recommendation 3 — Integrate Physical Security Into Endpoint Security Monitoring

Configure chassis intrusion detection on sensitive systems. Route chassis intrusion alerts and physical access control events to the SIEM. Establish response procedures for unscheduled physical access to sensitive endpoints.

**Priority:** Medium-High  
**Effort:** Medium  
**Detection improvement:** Meaningful — addresses the physical access precondition for most hardware-based attacks

---

### Recommendation 4 — Move Beyond Identity-Based USB Control

Augment VID/PID-based USB allowlisting with behavioral monitoring of USB device activity. Look for HID devices exhibiting anomalous input rates, devices that enumerate across unexpected device classes, or devices that appear and disappear in patterns inconsistent with normal user behavior.

**Priority:** Medium  
**Effort:** Medium (requires monitoring tooling capable of USB behavioral analysis)  
**Detection improvement:** Moderate — improves coverage for identity-spoofing scenarios

---

### Recommendation 5 — Include Hardware Attack Surface in Threat Models

Ensure that threat models for sensitive endpoints explicitly address hardware-level attack surfaces, including PCIe expansion card access, USB HID device spoofing, and physical access scenarios. Hardware threat categories are sometimes absent from threat models that focus primarily on network-borne and software-based threats.

**Priority:** Medium  
**Effort:** Low (documentation and review effort)  
**Detection improvement:** Indirect — improves security architecture and detection coverage decisions

---

### Recommendation 6 — Train SOC Analysts on Hardware-Layer Threat Recognition

Analysts should be able to recognize the signatures of hardware-related security events in endpoint telemetry: unexpected device enumeration, unusual driver loads, chassis intrusion alerts, and anomalous USB device behavior. This awareness improves alert triage accuracy and reduces the risk of hardware-related incidents being miscategorized or dismissed.

**Priority:** Medium  
**Effort:** Low (training and awareness program)  
**Detection improvement:** Meaningful — improves analyst capability to recognize and escalate hardware-related signals

---

### Recommendation 7 — Align Hardware Threat Detection to MITRE ATT&CK Framework

Map hardware-based threat detection rules and monitoring coverage to the relevant MITRE ATT&CK techniques. This provides a standardized, communicable framework for documenting detection coverage and gaps — directly useful for SOC reporting, purple team exercises, and security architecture reviews.

**Key techniques to align:**

| ATT&CK Technique | ID | Detection Data Source |
|---|---|---|
| Hardware Additions | T1200 | Asset management baselines; Sysmon Event ID 6; device enumeration logs |
| Input Capture: Hardware | T1056.002 | USB device behavioral monitoring; HID device enumeration events |
| Pre-OS Boot: System Firmware | T1542.001 | BIOS/UEFI integrity monitoring; firmware management platform alerts |

**Priority:** Medium  
**Effort:** Low (mapping and documentation exercise)  
**Detection improvement:** Indirect but high value — establishes shared language for communicating hardware threat coverage across SOC, detection engineering, and security architecture teams

---

## Section 4: SOC Analyst Relevance Summary

| Skill Area | How This Research Applies |
|---|---|
| Threat Intelligence | Understanding hardware attack surfaces completes the threat model for endpoint security |
| Alert Triage | Recognizing device enumeration, driver load, and chassis intrusion events as potential hardware threat indicators |
| Detection Engineering | Understanding EDR observability limits; designing hardware-aware compensating detections |
| Incident Response | Knowing when hardware forensics or physical security response is warranted |
| Security Architecture | Informing IOMMU configuration, USB control policies, and physical security integration |
| Risk Communication | Articulating hardware-layer risks and detection gaps to stakeholders accurately |
| Frameworks & Standards | Mapping real-world hardware threats to MITRE ATT&CK T1200, T1056.002, T1542.001 |

---

*For the underlying technical analysis: see [`technical-breakdown.md`](technical-breakdown.md)*  
*For the ethics framework: see [`ethics-and-safety.md`](ethics-and-safety.md)*
