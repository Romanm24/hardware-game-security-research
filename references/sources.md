# References and Sources

## Research Bibliography

**Project:** Hardware-Based Game Security Research  
**Author:** Roman Mares — SOC Analyst in Training | U.S. Army Veteran

---

> **Note on Sources:** This document provides reference categories, foundational texts, and placeholder citations for sources consulted or relevant to this research. Where specific sources were used in the preparation of documentation, they are cited by category. Readers are encouraged to consult primary sources directly for authoritative detail.

---

## Category 1: DMA and Hardware Memory Access Security

### Foundational Academic and Technical References

- **[PLACEHOLDER]** Frisk, A. (2016). *PCILeech: Direct Memory Access Attack Software.* GitHub repository. [Tool documentation reviewed for defensive research purposes; original research context: hardware memory forensics]

- **[PLACEHOLDER]** Bock, K., et al. (2017). *SoK: "Plug & Pray" Today — Understanding USB Insecurity in Versions 1 through C.* IEEE Symposium on Security and Privacy. [Establishes foundational framework for USB attack surface analysis]

- **[PLACEHOLDER]** Morgan, S. (2019). *IOMMU-based Protection Against DMA Attacks.* [Whitepaper / Technical Reference — specific publication details to be verified]

- **[PLACEHOLDER]** Intel Corporation. *Intel® Virtualization Technology for Directed I/O (Intel® VT-d): Architecture Specification.* Available from Intel Developer Documentation. [Authoritative reference for IOMMU implementation and DMA remapping]

- **[PLACEHOLDER]** Markettos, A. T., et al. (2019). *Thunderclap: Exploring Vulnerabilities in Operating System IOMMU Protection via DMA from Untrustworthy Peripherals.* Network and Distributed Systems Security (NDSS) Symposium. [Peer-reviewed research on IOMMU bypass vulnerabilities]

---

## Category 2: USB Security and Input Device Attack Surface

### USB Protocol and HID Security

- **[PLACEHOLDER]** Nohl, K., & Lell, J. (2014). *BadUSB — On Accessories that Turn Evil.* Presentation at Black Hat USA 2014. [Foundational research establishing USB firmware-level attack surface; widely cited in security literature]

- **[PLACEHOLDER]** United States Computer Emergency Readiness Team (US-CERT). *Security Tip: Using Caution with USB Drives.* CISA.gov. [Government advisory on USB-based threats; practical defensive guidance]

- **[PLACEHOLDER]** USB Implementers Forum (USB-IF). *Universal Serial Bus Specification.* [Authoritative protocol documentation for understanding device enumeration and HID class behavior]

- **[PLACEHOLDER]** Nissim, N., et al. (2017). *USB-Based Attacks.* Computers & Security, Elsevier. [Peer-reviewed survey of USB attack vectors and mitigations]

---

## Category 3: Anti-Cheat Systems and Game Security

### Game Security Architecture and Research

- **[PLACEHOLDER]** Johansson, E. (2018). *The State of Game Hacking and Anti-Cheat.* [Conference presentation — specific venue to be verified; overview of gaming security arms race from industry perspective]

- **[PLACEHOLDER]** Valve Corporation. *VAC (Valve Anti-Cheat): Technical Overview.* Valve Developer Community Documentation. [Industry documentation on software-based anti-cheat architecture and detection approach]

- **[PLACEHOLDER]** Easy Anti-Cheat / Epic Games. *Easy Anti-Cheat — Technical Whitepaper.* [Industry documentation; describes kernel-level detection approach and scope of software-based detection]

- **[PLACEHOLDER]** BattlEye. *BattlEye Anti-Cheat: Overview.* BattlEye.com. [Industry documentation; kernel-level detection architecture]

- **[PLACEHOLDER]** Wired / Ars Technica / PC Gamer. *[Multiple investigative articles on the evolution of hardware-based cheating in online games, 2020–2024.]* [Journalism establishing real-world context for hardware-based threat activity in gaming]

---

## Category 4: Endpoint Security and SOC Operations

### Endpoint Detection and Defensive Frameworks

- **[PLACEHOLDER]** MITRE ATT&CK Framework. *Technique T1200: Hardware Additions.* MITRE.org. [Standard threat classification framework; directly applicable to hardware-level attack surface documentation]

- **[PLACEHOLDER]** MITRE ATT&CK Framework. *Technique T1056.002: Input Capture — Hardware.* MITRE.org. [Standard threat classification for hardware-based input interception]

- **[PLACEHOLDER]** National Institute of Standards and Technology (NIST). *NIST Special Publication 800-53 Rev. 5: Security and Privacy Controls for Information Systems and Organizations.* [Authoritative framework for security control selection, including physical protection and media protection control families]

- **[PLACEHOLDER]** Center for Internet Security (CIS). *CIS Controls Version 8.* CIS-Security.org. [Control 10 (Malware Defenses), Control 3 (Data Protection), Control 12 (Network Infrastructure Management) — applicable to endpoint hardening recommendations]

- **[PLACEHOLDER]** NIST. *Special Publication 800-167: Guide to Application Whitelisting.* [Reference for device control and allowlisting policy design]

---

## Category 5: Hardware Security and Physical Access Threats

### Hardware-Level Security Research

- **[PLACEHOLDER]** Schneier, B. (2015). *Data and Goliath: The Hidden Battles to Collect Your Data and Control Your World.* W.W. Norton. [Broader context on hardware trust and physical access as security boundary]

- **[PLACEHOLDER]** Goodspeed, T., et al. *[Multiple publications on hardware debugging interfaces and physical access attack surfaces.]* Available via academic databases. [Foundational research on hardware as an attack surface]

- **[PLACEHOLDER]** NSA/CISA. *Hardware and Firmware Security Guidance.* CISA.gov. [U.S. Government guidance on firmware integrity, hardware attestation, and physical security integration]

- **[PLACEHOLDER]** Trusted Computing Group. *TPM Main Specification.* TrustedComputingGroup.org. [Reference for hardware attestation and platform integrity verification standards]

---

## Category 6: Research Ethics in Cybersecurity

### Ethical Frameworks

- **[PLACEHOLDER]** Kenneally, E., & Bailey, M. (2014). *Cyber-Security Research Ethics Dialogue and Strategy Workshop (CREDS): Workshop Report.* USENIX. [Framework for ethical cybersecurity research practice]

- **[PLACEHOLDER]** The Menlo Report: Ethical Principles Guiding Information and Communication Technology Research. (2012). U.S. Department of Homeland Security. [Foundational ethics framework for ICT security research]

- **[PLACEHOLDER]** ISC². *ISC² Code of Ethics.* ISC2.org. [Professional ethics framework for certified cybersecurity practitioners]

- **[PLACEHOLDER]** EC-Council. *Code of Ethics for Certified Ethical Hackers.* EC-Council.org. [Professional ethics standard for ethical security research]

---

## Notes on Citation Completeness

All citations marked **[PLACEHOLDER]** indicate sources where the reference category and topic are confirmed accurate but where specific publication details (volume, page numbers, DOIs, URLs) should be verified and updated before the repository is considered final for formal academic or professional submission.

This approach reflects responsible research documentation practice: establishing the relevant literature landscape accurately while acknowledging that specific bibliographic details require verification.

Readers conducting their own research on these topics are encouraged to:

1. Search USENIX, IEEE, and ACM digital libraries for peer-reviewed hardware security research
2. Consult MITRE ATT&CK and CAPEC for standardized threat categorization
3. Review NIST, CIS, and CISA publications for authoritative defensive guidance
4. Consult industry security research blogs from vendors such as CrowdStrike, Mandiant, and SentinelOne for contemporary threat intelligence on hardware-based threats

---

*Last updated: 2025*  
*For questions about source selection or citation accuracy, open a GitHub Issue.*
