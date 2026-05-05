# Technical Breakdown

## Hardware-Level Attack Surface Analysis — A Defensive Perspective

**Author:** Roman Mares — SOC Analyst in Training | U.S. Army Veteran  
**Classification:** Educational / Defensive Research  
**Scope:** Conceptual analysis only — no operational instructions included

---

## Introduction

This document provides a defense-oriented technical analysis of hardware-based security threats observed during this research project. The goal is to equip security practitioners — particularly SOC analysts and endpoint security engineers — with a conceptual understanding of how hardware-level threats undermine software-based security assumptions, and where defensive controls can be meaningfully applied.

No instructions for using, configuring, or deploying any hardware attack tool are included. All analysis is framed around understanding the threat in order to detect and respond to it.

---

## Section 1: The Hardware-Based Attack Surface

### 1.1 Understanding the Trust Hierarchy

Modern computing systems operate on an implicit trust hierarchy:

Security software — including anti-cheat systems, EDR agents, and application-layer monitoring tools — typically operates at the **application layer** or, in more privileged implementations, at the **OS kernel layer**. This means that software security controls are only as reliable as the hardware and firmware layers beneath them.

When an adversary introduces malicious or manipulated hardware into this stack, the fundamental trust assumptions of every layer above it become unreliable. This is the core challenge of hardware-based attacks.

### 1.2 PCIe as an Attack Surface

The Peripheral Component Interconnect Express (PCIe) bus is the primary high-speed interconnect used by modern computers to communicate with expansion cards — graphics cards, network adapters, storage controllers, and others.

PCIe devices, once initialized by the host system, are granted a level of system access governed by hardware mechanisms (IOMMU, DMA remapping) rather than by OS software policies. In systems where these hardware protections are not fully implemented, enabled, or enforced, a PCIe device can interact with system memory in ways that the OS cannot monitor or prevent through software controls alone.

**Defensive considerations for PCIe:**

- **IOMMU (Input-Output Memory Management Unit):** When properly enabled, IOMMU restricts which memory regions a PCIe device can access. Organizations should audit IOMMU configuration on sensitive endpoints.
- **Physical access controls:** PCIe attacks require physical access to the target machine. Physical security — locked cases, chassis intrusion detection, restricted physical access to systems — is a first-line control.
- **Firmware integrity monitoring:** BIOS/UEFI settings that govern PCIe access should be protected by strong firmware passwords and monitored for unauthorized changes.
- **Endpoint Detection and Response (EDR):** Some EDR solutions can flag unusual driver loads or device enumeration events that may indicate the introduction of unexpected PCIe hardware.

### 1.4 IOMMU Configuration and Auditing

The IOMMU is the primary hardware-level control against unauthorized PCIe memory access. Verifying its status is a foundational step in endpoint hardening.

**What to verify in BIOS/UEFI:**
- Intel systems: look for **VT-d (Virtualization Technology for Directed I/O)** — must be explicitly enabled
- AMD systems: look for **AMD-Vi** or **IOMMU** — must be explicitly enabled
- This setting is not enabled by default on all consumer or workstation motherboards — it must be audited, not assumed

**Verifying enforcement at the OS level:**

On Windows:

On Linux:
```bash
dmesg | grep -i iommu
dmesg | grep -i "DMAR: IOMMU"
```

**SOC analyst audit checklist:**
- Include IOMMU enablement status in endpoint hardening baselines
- Flag any endpoint where IOMMU is disabled as an elevated hardware risk
- Monitor BIOS/UEFI change logs where firmware management platforms expose them
- Treat IOMMU-disabled endpoints as having a reduced hardware trust boundary — document the risk and remediate where possible

---

### 1.5 EDR Visibility Into PCIe Device Activity

Understanding what EDR tools can and cannot see at the hardware layer is essential for accurate detection coverage assessment.

**What EDR tools can detect:**
- New device driver installations triggered by PCIe hardware introduction
- Unsigned or anomalous kernel driver loads
- OS-level device enumeration events (new hardware detected by the OS)
- Changes to the PCI device list surfaced through asset management integrations

**What EDR tools generally cannot detect:**
- Raw DMA memory reads on systems without IOMMU enforcement — these occur below the OS observability layer and produce no software-visible log entries
- PCIe device behavior that does not trigger a driver installation event
- Hardware memory interactions after initial device enumeration — subsequent DMA activity is not logged by standard EDR telemetry

**The structural boundary:**
EDR solutions monitor software-observable events. Activity that occurs entirely at the hardware layer — below the OS — does not produce software events and is therefore outside EDR's visibility window. This is a structural limitation of software-based monitoring, not a deficiency of any specific product.

**Recommended EDR configurations for hardware threat awareness:**
- Enable driver load event logging (Sysmon Event ID 6 on Windows)
- Alert on unsigned or low-prevalence driver installations
- Maintain a documented baseline of expected PCI device identifiers per endpoint class and alert on deviations from that baseline

---

### 1.6 Physical Security Controls for PCIe-Based Threats

Because PCIe-based hardware attacks require physical access to the target system, physical security controls are first-line — not supplementary — defenses.

| Control | Description | Defensive Value |
|---|---|---|
| **Chassis intrusion detection** | Sensors that log or alert when a system case is opened | Detects the physical access event that precedes PCIe hardware introduction |
| **Hardware asset inventory** | Documented baseline of all PCIe devices per endpoint | Enables detection of unauthorized hardware additions at next audit or scan |
| **Case locks / tamper-evident seals** | Physical deterrent and tamper indicator | Raises the effort bar; provides forensic evidence of unauthorized access |
| **Restricted physical access** | Badge-controlled rooms, locked racks, camera coverage | Reduces opportunity for unauthorized physical access to endpoints |
| **BIOS/UEFI passwords** | Prevents unauthorized firmware reconfiguration | Protects IOMMU and Secure Boot settings from being disabled by an attacker with brief physical access |
| **Secure Boot enforcement** | Ensures only signed firmware and bootloaders execute | Reduces firmware-layer attack surface adjacent to PCIe initialization |

**SOC integration note:** Chassis intrusion sensor events and physical access control logs should be routed to the SIEM and correlated with subsequent endpoint anomalies. Unscheduled physical access to a sensitive endpoint followed by unusual driver activity is a meaningful detection signal.

---

### 1.7 MITRE ATT&CK Mapping

This research maps directly to established MITRE ATT&CK framework techniques, providing a standardized vocabulary for communicating these threats in a professional security context.

| Technique | ID | Relevance to This Research |
|---|---|---|
| Hardware Additions | T1200 | Adversaries introduce computer hardware to gain access or establish persistence; directly covers PCIe DMA devices and input emulation hardware |
| Input Capture: Hardware | T1056.002 | Hardware-based input interception and emulation devices |
| Pre-OS Boot: System Firmware | T1542.001 | Firmware-level persistence; relevant to BIOS/UEFI security controls protecting IOMMU configuration |
| Exploitation for Defense Evasion | T1211 | Hardware-based techniques that operate below software detection layers |

**Recommended detection data sources per ATT&CK T1200:**
- Asset management systems maintaining PCIe device baselines
- Windows System Event Log — hardware device enumeration events
- Sysmon Event ID 6 — driver load events
- EDR telemetry — unsigned driver alerts
- Physical security system logs — chassis intrusion, badge access events

---

### 1.3 USB as an Attack Surface

The Universal Serial Bus (USB) protocol was designed for interoperability and ease of use — priorities that create inherent security trade-offs.

When a USB device is connected to a host, the host OS performs an enumeration process: it reads the device's self-reported identifiers (Vendor ID, Product ID, device class) and loads the appropriate driver. Critically, **the host OS has no cryptographic or hardware-level mechanism to verify that these self-reported identifiers are accurate.** The device can report any VID/PID it chooses.

This design limitation is well-known in security research and underlies a broad class of USB-based attacks, including:

- **BadUSB:** Exploiting the ability of USB devices to impersonate HID devices (keyboards, mice) to inject input
- **USB identity spoofing:** Devices that clone the VID/PID of trusted devices to evade allowlist-based USB device control policies
- **HID injection:** Synthesizing input events at the USB layer that the OS registers as legitimate human input

FUSER-class devices and KMBOX-type hardware operate within this same attack surface, applying these well-documented principles in gaming contexts.

**Defensive considerations for USB:**

- **USB device control policies:** Implement allowlisting based on device class, not just VID/PID, with the understanding that class-based controls are also bypassable with sufficient hardware capability
- **Physical port disabling:** On sensitive endpoints, physically disable or epoxy-fill unused USB ports
- **USB behavioral monitoring:** Rather than relying solely on device identity, monitor for behavioral anomalies — a "keyboard" that sends input events at machine-speed, or a device that enumerates rapidly across multiple device classes
- **OS-level USB restrictions:** Group Policy, endpoint management platforms, and OS-native controls can restrict which device classes are permitted to connect

---

## Section 2: Input Emulation — Security Risks and Detection Challenges

### 2.1 How Software Validates Input

Anti-cheat systems and other input-validation security software attempt to distinguish human input from automated or synthesized input using a range of behavioral heuristics:

- **Timing analysis:** Human input has natural variation in timing between keystrokes and mouse movements. Software may flag input patterns that are too regular or too fast for human generation.
- **Trajectory analysis:** Human mouse movement follows organic, slightly irregular paths. Perfect geometric trajectories may trigger behavioral flags.
- **Input rate limiting:** Certain actions performed at rates exceeding human physical capability may be flagged.

These heuristics are effective against **software-based** input automation running on the same host system, where the security software can observe the full input stack and compare what the application receives against what the OS reports.

### 2.2 Where Hardware Emulation Breaks Software Heuristics

Hardware input emulation devices — like the KMBOX category — operate at a fundamentally different layer. They generate USB HID events that the OS receives and processes as it would any legitimate peripheral input. The OS presents these inputs to applications — including security software — through its standard input APIs.

From the perspective of any software process on the host, including kernel-level security drivers, the input originated from a USB HID device and was processed through normal OS input pathways. The software has no visibility into whether that USB device was a physical keyboard operated by a human, or a hardware intermediary synthesizing inputs.

This is not a flaw in any specific anti-cheat implementation — it is a structural limitation of the software-hardware trust boundary.

### 2.3 Implications for Detection Engineering

For a SOC analyst or detection engineer, the key insight is:

> **Behavioral heuristics designed to detect software-based automation cannot reliably detect hardware-based input emulation.**

Detection strategies must therefore look for different signals:

- **Device enumeration anomalies:** Unexpected new USB devices appearing on endpoints, particularly devices that enumerate as HID but have no corresponding physical peripheral
- **USB topology changes:** HID devices that appear and disappear rapidly, or that change identity across sessions
- **System-level artifacts:** Unusual driver loads, unexpected USB device entries in system logs, or device manager events that correspond to hardware not physically present in the system's expected inventory
- **Physical security indicators:** Any indication of physical access to a device — chassis intrusion sensor events, unexpected physical presence

---

## Section 3: Memory Access Concerns

### 3.1 OS Memory Isolation — What It Protects Against

Modern operating systems implement robust process memory isolation. Each process operates in its own virtual address space. One process cannot read or modify the memory of another process without explicit OS-mediated mechanisms (shared memory, inter-process communication APIs) that require appropriate permissions.

This isolation is enforced by the OS kernel and the CPU's memory management unit (MMU). It is highly reliable against software-based attacks: a malicious process running on a system generally cannot read the memory of a protected process.

### 3.2 Where Hardware Access Changes the Equation

DMA-capable hardware does not access memory through the OS's virtual memory abstraction. It communicates with physical memory through the system's memory controller. Where hardware-level memory protections (specifically, IOMMU-based DMA remapping) are absent or misconfigured, this access is not subject to the OS's process isolation model.

This means that security software relying on OS process isolation as a protection mechanism faces a structural gap: the isolation it enforces is reliable against software-level threats but does not extend to hardware-level memory access.

### 3.3 IOMMU as a Hardware Defense

The IOMMU is the hardware-level counterpart to the CPU's MMU — a component designed specifically to provide memory isolation for DMA-capable devices. When properly configured, IOMMU restricts each PCIe device to accessing only the memory regions it has been authorized to access, preventing arbitrary DMA reads and writes to protected memory.

**IOMMU-related defensive recommendations:**

- Verify that IOMMU is enabled in BIOS/UEFI settings on sensitive endpoints
- Audit OS-level IOMMU enforcement (Intel VT-d, AMD-Vi settings)
- Review security advisories related to IOMMU bypass vulnerabilities, which are periodically discovered and patched
- Include IOMMU configuration status in endpoint hardening checklists

---

## Section 4: Endpoint Trust — Systemic Gaps

### 4.1 The Assumption of Trusted Hardware

The fundamental security gap documented in this research can be stated concisely:

> **Software security systems are designed to operate on a trusted hardware foundation. When that foundation is compromised or manipulated, software security controls may fail silently — producing no alerts, no logs, and no indication of compromise.**

This is not a unique observation. It is a well-established principle in hardware security research, supply chain security, and physical security doctrine. This project applies it to the specific context of gaming endpoint security to illustrate how the principle manifests in a consumer-facing environment.

### 4.2 Implications for Security Architecture

For security architects and analysts, this research reinforces several foundational principles:

**Defense in Depth Is Not Optional**
No single security control is sufficient. Software-based endpoint security must be layered with physical security controls, hardware attestation, firmware integrity monitoring, and network-level behavioral monitoring.

**Physical Security Is a Cybersecurity Control**
Attacks that require physical hardware access underline the importance of physical security as a component of an integrated security program. Locked equipment rooms, chassis intrusion detection, hardware asset management, and visitor access controls are cybersecurity controls, not merely facilities concerns.

**Trust Verification Must Be Continuous**
Static trust decisions — "this endpoint is trusted because it passed an enrollment check" — are insufficient in environments where hardware state can change. Continuous verification of hardware configuration, unexpected device events, and behavioral anomalies is necessary for a robust security posture.

**Detection Must Account for Hardware Blind Spots**
Detection strategies that rely entirely on software-observable signals will have blind spots corresponding to hardware-layer activity. Effective detection programs acknowledge these gaps and supplement software monitoring with physical security controls and hardware attestation where high-value assets warrant it.

---

## Section 5: Summary Table — Threat, Gap, and Defensive Control

| Threat Category | Security Gap | Recommended Control |
|---|---|---|
| DMA PCIe Hardware | OS memory isolation does not apply to hardware DMA access | IOMMU enforcement; physical access controls; hardware asset management |
| Input Emulation (KMBOX) | Software input heuristics cannot distinguish hardware-synthesized HID events from human input | USB device behavioral monitoring; physical security; endpoint hardware inventory |
| USB Identity Spoofing (FUSER) | Host OS trusts self-reported USB device identifiers; no cryptographic verification | Device class-based controls; behavioral monitoring; physical port controls |
| General Hardware Manipulation | Software security controls assume trusted hardware layer | Defense in depth; physical security integration; hardware attestation; continuous monitoring |

---

*For research observations and specific defensive recommendations: see [`findings.md`](findings.md)*  
*For the full ethics framework governing this research: see [`ethics-and-safety.md`](ethics-and-safety.md)*
