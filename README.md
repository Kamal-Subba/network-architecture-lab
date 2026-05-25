# home-soc-lab
 
*(pfSense + Wazuh SIEM + VLAN Segmentation + Security Monitoring Lab)*
 
This repository documents the design, deployment, troubleshooting, hardening, and ongoing evolution of a self-hosted Security Operations Center (SOC) and segmented home lab environment.
 
The objective of this project is not simply to deploy security tools, but to understand how infrastructure behaves operationally and defensively under real conditions — including deployment failures, segmentation mistakes, recovery procedures, monitoring workflows, and infrastructure hardening decisions.
 
This environment is treated as a continuously evolving operational lab rather than a static showcase project.
 
The repository is used to document:
- Infrastructure architecture [Over View](architecture/README.md)
- Security design decisions
- Detection engineering [SIEM Deployment](detection/wazuh-deployment.md)
- Segmentation strategy
- Monitoring workflows
- Operational troubleshooting
- Incident-style recovery processes
- Adversary simulation exercises
- Lessons learned during deployment and maintenance
---
 
# Current Environment State
 
## Core Infrastructure
 
### Network & Routing
- pfSense deployed as primary firewall/router
- Managed VLAN-capable switching infrastructure operational
- Multi-VLAN network segmentation deployed using 802.1Q
- Dedicated management network architecture in progress
- Inter-VLAN firewall policy enforcement operational
- DHCP segmentation and reserved addressing strategy implemented
### Wireless Infrastructure
- AP controller deployed locally
- Multi-AP wireless mesh deployment operational
- Centrally managed SSIDs configured
- VLAN-aware wireless infrastructure operational
- Guest wireless segmentation deployed and actively tested
- Wireless isolation and segmentation validation in progress
### Monitoring & SIEM
- Wazuh SIEM deployed on dedicated Ubuntu infrastructure
- Multiple endpoints onboarded through Wazuh agents
- Centralized endpoint telemetry collection operational
- Endpoint visibility and monitoring workflows established
- Log aggregation and alert monitoring actively expanding
---
 
# Current Security Capabilities
 
## Implemented Controls
 
- Stateful inter-VLAN firewall policy enforcement
- VLAN-based network isolation
- Guest network internet-only restrictions
- Controller-managed wireless segmentation
- Centralized endpoint monitoring through Wazuh
- Dedicated infrastructure segmentation planning
- Restricted management-access philosophy
- UFW-based host firewall hardening
- Infrastructure IP management and DHCP reservation strategy
- Wireless isolation testing and validation
---
 
# Network Segmentation Model
 
## Current VLAN Structure
 
| VLAN    | Purpose                          |
|---------|----------------------------------|
| VLAN #  | Management / Infrastructure      |
| VLAN #  | Guest Wireless Network           |
| VLAN #  | SIEM / Monitoring Infrastructure |
 
Current segmentation is enforced through:
- pfSense firewall policy controls
- VLAN-aware switching
- Wireless VLAN mapping
- Layered infrastructure separation
---
 
# Planned Segmentation Expansion
 
| Planned Segment | Purpose                                 |
|-----------------|-----------------------------------------|
| RED             | Adversary simulation / offensive testing |
| BLUE            | Trusted internal systems                |
 
The long-term goal is to simulate a small enterprise-style segmented environment capable of supporting:
- Security monitoring
- Controlled attack simulation
- Detection validation
- Threat hunting workflows
- Incident investigation exercises
---
 
# Operational Philosophy
 
A major focus of this project is understanding infrastructure operationally — not just functionally.
 
This includes understanding:
- How systems communicate
- How segmentation fails
- How infrastructure recovers
- How telemetry is generated
- How alerts correlate across systems
- How management boundaries should be enforced
- How security controls affect operational usability
The environment is intentionally being developed with an emphasis on:
- Least privilege
- Segmentation
- Explicit trust boundaries
- Infrastructure visibility
- Operational resilience
- Detection-oriented architecture
---
 
# Detection & Monitoring Goals
 
Current and planned monitoring activities include:
 
- Wazuh alert monitoring and tuning
- Endpoint telemetry analysis
- Firewall log analysis
- Detection validation
- Security event correlation
- Threat hunting exercises
- Detection rule refinement
- Infrastructure anomaly identification
- Log enrichment workflows
- Adversary behavior simulation validation
---
 
# Red Team Simulation Scope
 
The offensive-testing side of the lab is intended strictly for isolated internal simulation and adversary emulation.
 
Planned simulation activities include:
- Network reconnaissance
- Service enumeration
- Port scanning
- Credential attack simulation
- Web application enumeration
- Simulated lateral movement
- Detection validation against attacker behavior
All testing is restricted to isolated lab segments.
 
---
 
# Failures & Recovery Documentation
 
One of the core goals of this repository is documenting not only successful deployments, but also operational failures and recovery processes.
 
Incidents are documented within their relevant infrastructure section rather than a single centralized folder, reflecting the nature of each failure.
 
Documented incidents:
- [VLAN Guest Network Outage — Incident Report & Recovery](architecture/incidents/vlan-guest-network-outage.md)
- [AP Adoption Failure — Troubleshooting & Recovery](wireless/incidents/omada-ap-adoption-troubleshooting.md)
Planned documentation areas:
- DHCP segmentation issues
- Management-access loss scenarios
- Segmentation validation failures
- pfSense firewall and routing incidents
The objective is to treat failures as operational learning opportunities and document root-cause analysis.
 
---
 
# Tools & Technologies
 
## Infrastructure
- pfSense — primary firewall, routing, and VLAN policy enforcement
- TP-Link Omada ecosystem — centrally managed wireless infrastructure
- Managed VLAN-capable switching — 802.1Q trunk and access port segmentation
- Ubuntu Server — Wazuh SIEM host and supporting infrastructure
- Intel NUC — dedicated lab compute infrastructure
## Security & Monitoring
- Wazuh SIEM — centralized endpoint telemetry, alerting, and detection
- UFW — host-based firewall hardening on Linux endpoints
- Syslog — infrastructure log forwarding and aggregation
- Endpoint telemetry collection — agent-based visibility across lab systems
## Operating Systems
- Ubuntu Linux
- Windows endpoints
- Kali Linux
## Supporting Services
- MongoDB — backend data store for Wazuh SIEM
- SSH-based administration
- Git/GitHub documentation workflows
---
 
# Repository Structure
 
```text
architecture/
  incidents/          Segmentation and design failure writeups
 
pfsense/
  incidents/          Firewall and routing incident documentation
 
switches/             VLAN and switching configuration documentation
 
wireless/
  incidents/          Wireless infrastructure incident documentation
 
detection/
  incidents/          SIEM and detection failure documentation
 
attack-simulations/   Attack simulation and adversary emulation
red-team/             Offensive testing notes and workflows
blue-team/            Defensive monitoring and investigation workflows
log-samples/          Sanitized logs and alert samples
```
 
---
 
# Current Development Focus
 
## Active Work
- Wazuh infrastructure expansion
- Additional endpoint onboarding
- pfSense log integration
- VLAN policy refinement
- Management network hardening
- Detection validation workflows
- Infrastructure access restriction
- Security monitoring improvements
## Upcoming Goals
- RED / BLUE segmented lab deployment
- Custom Wazuh detection rules
- Adversary emulation workflows
- Network traffic analysis pipelines
- Threat hunting exercises
- Detection enrichment and correlation
- Expanded infrastructure hardening
---
 
# Security Considerations
 
This environment is isolated and intended strictly for educational, research, and defensive security purposes.
 
All offensive testing is restricted to internally controlled lab environments.
 
The following are intentionally excluded or sanitized before publication:
- Credentials
- Internal IP schemes
- Public IP addresses
- API keys
- Personal identifiers
- Infrastructure-specific secrets
---
 
# Project Goal
 
The long-term objective of this lab is to build practical experience operating and securing segmented infrastructure environments while developing hands-on capability in:
 
- Security monitoring
- Detection engineering
- Incident investigation
- Infrastructure hardening
- Network security
- SIEM operations
- Adversary simulation
- Operational troubleshooting
- Defensive security architecture
This repository is intended to evolve alongside the environment itself and document that progression over time.
 
