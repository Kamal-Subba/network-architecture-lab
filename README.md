# Network Architecture Lab

This repository documents the network infrastructure lab I built to support future technical and cybersecurity projects.

The lab serves as the backbone for additional projects by providing a structured environment for network segmentation, firewall routing, wireless infrastructure, centralized logging, and operational testing.

This lab is designed to support isolated VLANs, log telemetry integration, guest network testing, troubleshooting scenarios, lessons learned documentation, and future scalability as the environment grows.

---

## Lab Focus

- Firewall routing and rule management using pfSense
- VLAN segmentation for separating trusted, guest, management, and testing networks
- Managed switching and wireless access point configuration
- Centralized log collection and monitoring using Wazuh
- Controlled troubleshooting and documentation of network issues
- Building a reusable foundation for future security, server, and infrastructure projects

---

## Core Infrastructure

### Network & Routing

- pfSense deployed as the primary firewall/router
- Managed VLAN-capable switching infrastructure operational
- Multi-VLAN segmentation implemented using 802.1Q
- Inter-VLAN firewall policy enforcement operational
- DHCP segmentation and addressing strategy implemented
- Dedicated management network architecture

### Wireless Infrastructure

- Centrally managed wireless access points configured
- VLAN-aware wireless infrastructure operational
- Guest wireless segmentation deployed and tested
- Wireless isolation and segmentation validation in progress

### Monitoring & Logging

- Wazuh deployed on dedicated Ubuntu infrastructure
- Multiple endpoints onboarded through Wazuh agents
- Centralized endpoint telemetry collection operational
- pfSense log forwarding integration in progress / operational
- Log aggregation and monitoring workflows actively expanding

---

## Network Segmentation Model

| Segment | Purpose |
|---|---|
| Management / Infrastructure | Administrative access and infrastructure management |
| Guest Wireless | Internet-only guest access |
| Monitoring | SIEM and logging infrastructure |
| Vulnerable Devices | isolated testing systems |
| Threat Simulation | controlled testing systems |

Current segmentation is enforced through:

- pfSense firewall policy controls
- VLAN-aware switching
- Wireless VLAN mapping
- Layered infrastructure separation

---

## Operational Philosophy

A major focus of this project is understanding infrastructure operationally, not just functionally.

This includes understanding:

- How systems communicate
- How segmentation is enforced
- How segmentation can fail
- How infrastructure recovers from misconfiguration
- How telemetry is generated
- How management boundaries should be protected
- How security controls affect usability

The environment is intentionally being developed with an emphasis on:

- Least privilege
- Network segmentation
- Explicit trust boundaries
- Infrastructure visibility
- Operational resilience
- Documentation-driven learning

---

## Recovery Documentation

One of my goals for this repository is documenting not only successful deployments, but also operational failures and recovery processes.

Documented incidents:

- [VLAN Guest Network Outage — Incident Report & Recovery](incidents/vlan-misconfig-network-outage.md)
- [AP Adoption Failure — Troubleshooting & Recovery](incidents/omada-ap-adoption-troubleshooting.md)

The objective is to treat failures as operational learning opportunities and document root-cause analysis.

---

## Tools & Technologies

### Infrastructure

- pfSense - primary firewall, routing, and VLAN policy enforcement
- AP Controller- Centrally managed wireless infrastructure
- Managed VLAN-capable switch — 802.1Q trunk and access port segmentation
- Ubuntu Server - dedicated lab server infrastructure
- Intel NUC - dedicated lab compute hardware
 
### Monitoring & Administration

- Wazuh — centralized endpoint telemetry, alerting, and monitoring
- UFW — host-based firewall hardening on Linux systems
- Syslog — infrastructure log forwarding and aggregation
- SSH — remote system administration
- Git/GitHub — documentation and version control

### Operating Systems

- Ubuntu Linux
- Windows endpoints
- Kali Linux

### Supporting Services

- Omada Controller
- MongoDB
- OpenJDK
- DHCP
- DNS
- Syslog

---

## Repository Structure

```text
architecture/          Network design, diagrams, segmentation model, and traffic flows

infrastructure/        pfSense, switching, wireless, Wazuh, and endpoint infrastructure

configurations/        Configuration design notes for VLANs, firewall rules, DHCP, logging, and access control

operations/            Maintenance notes, troubleshooting steps, backups, recovery, and operational lessons learned

incidents/             Infrastructure and network issue writeups, root-cause analysis, and lessons learned

assets/                Sanitized diagrams, screenshots, and supporting images

```

---

*This is an isolated lab environment built exclusively for educational, testing, and defensive security purposes. Any testing is performed only within controlled systems owned and managed in the lab.*

