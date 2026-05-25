## Wazuh SIEM Deployment

## Overview

This document is to cover the deployment of dedicated Wazuh SIEM in my home lab environment.
The objective was to transition from a temporary laptop-based setup to a dedicated, remotely managed security monitoring appliance capable of centralized log aggregation and firewall telemetry ingestion. 

This is a **living document** updated as the lab evolves. Detection engineering, log analysis, and attack simulation are covered in the companion **Security Operations document**.

## Deployment Focus

- Operational stability
- Centeralized logging
- Remote administration 
- Incremental infrastructure hardening

The Wazuh server is deployed on a dedicated NUC running Ubuntu server and integrated with pfsense remote syslog forwarding. 


---


## Objectives
 
| Goal | Status |
|---|---|
| Deploy dedicated SIEM infrastructure | ✅ Complete |
| Centralize firewall telemetry (pfSense) | ✅ Complete |
| Enable persistent, always-on monitoring | ✅ Complete |
| Transition to headless server administration | ✅ Complete |
| Build scalable foundation for endpoint onboarding | ✅ Complete |

*(purposely not including future goals at current state)*


## Hardware used
 
I repurposed an existing NUC from my lab environment for the dedicated SIEM deployment. Although it was not originally intended to serve as a SIEM host, its available resources were sufficient for the current workload after reviewing Wazuh sizing expectations and observing real-world utilization.

**Specifications:**
- RAM: 16GB
- Storage: SATA SSD 250GB

**Roles running on the node:**
- Wazuh Manager
- Wazuh Dashboard
- Wazuh Indexer

**Operational Observations:**
There were still some concerns i had as to whether the hardware could hold the workload, but everything has stayed stable even after integrating firewall logs. Here are the observed details: 

- Thermals remained stable throughout deployment and sustained operation 
- Resource utilization acceptable for small-scale SOC workloads
- OpenSearch RAM consumption is expected given it's Java-based indexing arahitecture and was accounted for in research. 

I was very concerned about the fan and how much noise it might make, but after few days of observation it runs near silent. 
The fan does spin up during peak log ingestion, but it's barely noticeable. Just the setting on a wooden surface can amplify the sound. 

---

## Operating System

**Platform:** Ubuntu Server

Installation decisions intentionally prioritized simplicity and recoverability over security hardening at this stage. Hardening is applied incrementally post-deployment to avoid troubleshooting complexity during baseline bring-up.

Few things i did while i was standing up the server. 
- Standard server installation - to reduce complexity during initial deployment. 
- OpenSSH enabled at install - wanted remote admin mgmt as soon as the server was up. 
- Ran Full disk and no LVM - to simplify partition recovery workflow
- No LUKS encryption - security hardening through local firewall and other methods, not likely my hardware will go missing. 

The server was also deployed on a flat VLAN during the initial deployment to establish a stable baseline before introducing network complexity to reduce the number of variables during troubleshooting. 
Once the deployment was stable it was segmented to VLAN.

---

## Initial System Config

### Baseline Validation Checklist
- [x] Network connectivity confirmed 
- [x] DNS resolution validated
- [x] System packages updated 
- [x] SSH access verified from mgmt system
- [x] Remote administration confirmed 
- [x] Resource utilization baseline recorded

### Utilities Installed 

```
curl | unzip | nano | net-tools | htop | ufw

```

After i validated SSH, it was transitioned to a full headless administration. _although i have connected my monitor and mouse from time to time just see it there_.
But most of the time i am working via SSH and HTTPS. 

---


## Wazuh Deployment
 
### Single-Node Installation
 
Wazuh was deployed using the official Wazuh installation assistant script.
 
**Components installed:**
- Wazuh Manager
- Wazuh Dashboard
- Wazuh Indexer (OpenSearch backend)
**Notable deployment considerations:**
 
- Ubuntu 24.04 LTS was not yet officially listed by the installer at time of deployment. Installer validation checks required bypassing to proceed. Compatibility was confirmed manually prior to bypassing.
- Installation time was dominated by OpenSearch initialization and initial index setup — expected behavior for this stack.
### Post-Installation Validation
 
- Service health confirmed via `systemctl`
- Dashboard accessibility verified
- Resource utilization monitored via `htop`
- Administrative credentials rotated (see Password Management)
- Service restart cycle validated after credential rotation

---


## Resource Monitoring

**Observations post-deployment:**

| Resource | Obsservation |
|---|---|
| RAM | OpenSearch allocated significant heap — expected, monitored |
| CPU | Spikes observed during indexing and post-update cache rebuild — normalized |
| Swap | No swap usage detected |
| Stability | System remained responsive under sustained load |


I've been using `htop` as my main monitoring tool, but it seemed a bit unreliable as times, especially after a `apt update && upgrade` recently. Htop reported CPU at or over 100% following the update. However, there were no other indicators suggested actual resource pressure. Leading me to rely on `landscape-sysinfo` for more constant baseline.
I've been using 'landscape-sysinfo' to get resource utilization info more accourately, but still depending on `htop` for pid's running.

*(Update)* After reviewing this further, it appears that Linux CPU usage can be represented per core, which can make `htop` display values above 100% on multi-core systems. In this case, the higher CPU readings did not necessarily indicate full system saturation.
I am continuing to use `htop` for identifying running processes, PIDs, and service-level CPU activity, while using `landscape-sysinfo` as a quick baseline reference for overall system health. 


## Password Management

Administrative credentials were rotated using Wazuh's built-in password management tooling immediately after initial deployment.
 
**Key operational notes:**
- Credential changes require service restarts across all three components
- Authentication is tightly coupled to the OpenSearch security plugin
- Manual editing of authentication config files was intentionally avoided to prevent configuration corruption
**Services restarted after credential rotation:**
```
wazuh-indexer
wazuh-manager
wazuh-dashboard
```
 
---
 
## pfSense Telemetry Integration
 
### Remote Syslog Forwarding
 
pfSense was configured to forward syslog telemetry to the Wazuh node over the network.
 
| Setting | Value |
|---|---|
| Transport | UPD  |
| port | | 514 |

**Log categories enabled:**
 
| Category | Rationale |
|---|---|
| Firewall Events | Core perimeter visibility; connection allow/deny telemetry |
| DHCP Events | Asset tracking; detects rogue or unexpected devices |
| System Events | Platform health; authentication and config change awareness |
 
Categories were deliberately scoped to high-value telemetry to minimize unnecessary log volume during initial deployment. Additional categories will be evaluated as detection tuning matures.
 
---
 
## rsyslog Integration
 
Ubuntu's rsyslog service was configured to listen for and receive the inbound pfSense syslog stream.
 
**Validation steps performed:**
- Packet capture confirmed traffic arriving on UDP/514
- Port listener confirmed active via `ss` / `netstat`
- Live log monitoring confirmed data was being written
---
 
## Dedicated Log Routing
 
A dedicated rsyslog rule was created to route pfSense telemetry into a **separate log file**, isolated from general Ubuntu system logs.
 
**Benefits:**
- Cleaner ingestion pipeline into Wazuh
- Easier manual troubleshooting during initial setup
- Improved separation of firewall telemetry from host OS events
- Better foundation for future detection rule tuning
---
 
## Host Firewall Hardening (UFW)
 
Basic host-level firewall restrictions were implemented using UFW following the principle of least privilege.
 
**Current ruleset:**
 
| Service | Restriction |
|---|---|
| SSH (port 22) | Limited to dedicated management system only |
| Wazuh Dashboard (port 443) | Restricted to trusted systems |
| Syslog ingestion (UDP 514) | Restricted to approved source devices |
 
**Planned additions:**
- Wazuh agent enrollment port restrictions
- Endpoint agent communication path controls
- Inter-VLAN access controls post-segmentation
---
 
## Operational Lessons Learned
 
These reflect real decisions and tradeoffs encountered during deployment — not theoretical best practices.
 
- **Sequence matters.** Establish a stable baseline on flat networking before introducing VLAN segmentation. Reducing variables during bring-up saves significant troubleshooting time.
- **Validate at every layer.** Packet capture is faster than log chasing when diagnosing ingestion issues. Confirm traffic is arriving before debugging the application layer.
- **Credential management requires a restart plan.** OpenSearch's tight coupling to the security plugin means credential rotation is a multi-service operation. Plan accordingly.
- **Incremental hardening prevents lockouts.** Apply firewall rules in stages. Validate access after each rule addition before proceeding.
- **Don't over-optimize stable infrastructure.** OpenSearch resource usage looks alarming at first glance — it's normal. Resist the urge to tune aggressively before establishing a performance baseline.
- **Headless administration improves workflow.** Committing to SSH-only administration from day one builds better operational habits.
---
 
## Current State
 
| Area | Status |
|---|---|
| Wazuh platform | ✅ Operational |
| Remote administration | ✅ Headless via SSH |
| pfSense telemetry ingestion | ✅ Active |
| Resource stability | ✅ Confirmed |
| Host firewall hardening | ✅ Applied |
 
---
 
## Related Documentation
 
| Document | Description |
|---|---|
| Security Operations (coming soon) | Log review, detection findings, alert triage, and attack simulation |
 

