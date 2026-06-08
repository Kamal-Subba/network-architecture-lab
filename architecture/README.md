# Network Security Architecture
 
## Overview

This doc will go over the logical and security archecture of a segmented network env designe to support secure connection, traffic isolation and centeralized security monitoring.  
The design follows a firewall-centric enforcement model with explicit trust zone separation and controlled inter-zone communication. It is intended to simulate enterprise network segmentation patterns used in security operations environments, including defensive monitoring and controlled adversary simulation zones.
 
---
 
## Logical Lab Overview
 
```
                          [ INTERNET ]
                               |
                               |
                      +--------+--------+
                      |   pfSense       |
                      |  Edge Firewall  |
                      |  (NAT / Policy) |
                      +--------+--------+
                               |
                    (802.1Q Trunk Link)
                               |
                      +--------+--------+
                      |  Managed L2     |
                      |    Switch       |
                      +---+---+---+-----+
                          |   |   |   |
           +--------------+   |   |   +------------------+
           |                  |   |                      |
    +------+------+    +------+------+          +--------+------+
    |  MGMT NET   |    |  USER NET   |          |  GUEST NET    |
    |  VLAN #     |    |  (planned)  |          |  VLAN #       |
    |             |    |             |          |               |
    | - pfSense   |    | - Trusted   |          | - Internet    |
    | - Switch    |    |   Clients   |          |   only        |
    | - APs       |    |             |          | - No internal |
    | - NUCs      |    |             |          |   access      |
    +-------------+    +-------------+          +---------------+
 
           +--------------+        +------------------+
           |              |        |                  |
    +------+------+    +--+--------+----+
    |  Blue NET   |    |    Red Net     |
    |  VLAN #     |    |    VLAN #	|
    |             |    |                |
    | - Wazuh     |    | - Kali Linux   |
    |   SIEM      |    | - Attack sim   |
    | - Log       |    | - Fully        |
    |   ingestion |    |   isolated     |
    | - Detection |    |                |
    +-------------+    +----------------+
 
  Telemetry Flow (push-based, one-directional):
  USER NET  ──logs──►  BLUE NET (Wazuh)
  MGMT NET  ──logs──►  BLUE NET (Wazuh)
  pfSense   ──logs──►  BLUE NET (Wazuh)
```
 
---
 
## Core Components
 
| Component | Role |
|-----------|------|
| **pfSense (Edge Firewall)** | Central routing and policy enforcement. Handles NAT, inter-VLAN routing, and stateful firewall rules. Default-deny posture between all zones. |
| **Managed Layer 2 Switch (802.1Q)** | VLAN segmentation and trunk distribution. Connects firewall, access points, and endpoints. Enforces logical separation at Layer 2. |
| **TP-Link Omada Wireless APs** | Wireless connectivity mapped to VLAN-backed SSIDs. Supports internal and guest wireless segmentation. |
| **Ubuntu Server (Intel NUC)** | Dedicated Wazuh SIEM host. Isolated on B NET for analytical monitoring. |
 
---
 
## Network Segmentation Model
 
The environment is divided into logically isolated security zones enforced through pfSense VLAN policy:
 
| Zone | VLAN | Purpose | Trust Level | Status |
|------|------|---------|-------------|--------|
| MGMT NET | # | Infrastructure administration — access restricted to dedicated hardwired infrastructure | Restricted | Operational |
| GUEST NET| # | Internet-only access, fully isolated from internal infrastructure | Untrusted | Operational |
| BLUE NET | # | Defensive monitoring, Wazuh SIEM, detection engineering | Analytical | Operational |
| USER NET | TBD | General-purpose trusted client devices | Trusted | Planned |
| RED NET  | TBD | Controlled offensive testing and adversary emulation | Isolated | Planned |
 
---
 
## Traffic Flow and Security Policy Matrix
 
All inter-zone traffic is denied by default. Permitted flows are explicitly defined through pfSense firewall rules.
 
| Source Zone | Destination Zone | Permitted | Protocol / Port | Purpose |
|-------------|-----------------|-----------|-----------------|---------|
| MGMT NET | All Zones | Limited | HTTPS / SSH | Infrastructure administration |
| USER NET | Internet | Yes | HTTP / HTTPS | General client access |
| USER NET | MGMT NET | No | — | Prevent administrative exposure |
| GUEST NET | Internet | Yes | HTTP / HTTPS | Isolated internet access |
| GUEST NET | Any Internal Zone | No | — | Prevent lateral movement |
| RED NET | USER NET | No | — | Attack simulation isolation |
| RED NET | MGMT NET | No | — | Prevent infrastructure compromise |
| BLUE NET | All Zones | Read-only | Syslog / Agent | Log ingestion only — no active polling |
 
### Enforcement Notes
 
- All inter-VLAN routing is handled exclusively by pfSense
- No direct Layer 2 communication exists between VLANs
- All cross-zone traffic is validated at the firewall layer before forwarding

---
 
## BLUE NET Telemetry Model
 
BLUE NET does not initiate connections into other security zones. It operates on a **push-based telemetry model** — log sources forward data outbound to Wazuh rather than being polled from the SIEM.
 
This ensures analytical visibility across all zones without requiring inbound firewall rules that would expose the monitoring infrastructure.
 
| Log Source | Forwarding Method |
|------------|-------------------|
| pfSense firewall | Syslog export via internal logging configuration |
| Linux endpoints | Wazuh agent (outbound to BLUE NET) |
| Windows endpoints | Wazuh agent (outbound to BLUE NET) |
| Switches / APs | Management-plane syslog where supported |
 
---
 
## Security Telemetry Sources (Wazuh SIEM)
 
### Firewall Telemetry (pfSense)
- Allow / deny log events
- NAT and port forwarding activity
- Inter-VLAN routing attempts
- IDS/IPS events (planned — Suricata integration)
### Endpoint Telemetry (Wazuh Agents)
- Authentication events (success and failure)
- File integrity monitoring (FIM) on critical paths
- Process execution logs
- Windows Event Logs and Linux auth logs
### Infrastructure Logs
- Switch port status and link state changes
- VLAN assignment changes where supported
- AP association and disassociation events
### Access and Authentication Events
- Administrative login attempts to pfSense
- SSH and management interface access
- Failed authentication attempts across monitored systems
---
 
## Active Detection Use Cases
 
| Detection Use Case | Data Source | Method |
|--------------------|-------------|--------|
| Failed authentication | Endpoint agents, pfSense | Rule-based alerting on repeated failures |
| Inter-VLAN policy violations | pfSense firewall logs | Alert on denied cross-zone traffic attempts |
| File integrity violations | Wazuh FIM | Alert on modification of monitored critical paths |
| Lateral movement indicators | Firewall logs | Anomalous traffic patterns from RED / GUEST zones |
| Privilege escalation signals | Endpoint process and auth logs | Correlation of suspicious privilege changes |
 
---
 
## Design Principles
 
- **Default-deny posture** — All inter-zone traffic is blocked unless explicitly permitted
- **Trust zone segmentation** — Zones are separated based on operational risk and function
- **Centralized policy enforcement** — pfSense is the single enforcement point for all inter-VLAN traffic
- **Push-based telemetry** — Monitoring is passive; no active polling from BLUE NET into production zones
- **Offensive / defensive separation** — RED NET is physically and logically isolated from all other zones
- **Least privilege access** — Management access is restricted to MGMT NET only
---
 
 
 
