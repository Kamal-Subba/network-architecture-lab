# VLAN Design

## Purpose

The VLAN design separates devices by role, trust level, and intended function within the lab.
- Support controlled testing environments where activity can be isolated, logged, and analyzed.
- Learn first hand what/how the systems needs to be configured and functions.
- Reduce exposure between systems and have cleaner way to follow traffics back when troubleshooting. 


## VLAN Overview

| VLAN | Segment Name | Purpose | Status |
|---|---|---|---|
| VLAN # | Trusted LAN | Main trusted/internal network | Active |
| VLAN # | Guest Network | Guest wireless and guest testing | Active |
| VLAN # | Management | Admin access to infrastructure | Active |
| VLAN # | Monitoring | Wazuh/logging infrastructure | Active |
| VLAN # | Testing | Isolated testing network | Active |
| VLAN # | Lab Services | Future internal lab services | Planned |

## Segmentation Logic

The VLAN design is based on separating devices by function and trust level.
Trusted/internal devices are kept separate from guest and future testing networks unless specific traffic is intentionally allowed for a documented project or service requirement.
It's designed this way to support security monitoring and testing while reducing unnecessary access between network segments.

## Traffic Control Model

Inter-VLAN traffic is controlled through pfSense firewall rules.

The default design approach is:

- Allow only required traffic between VLANs
- Deny unnecessary inter-VLAN communication by default
- Block guest access to internal infrastructure
- Restrict management access to trusted administrative systems
- Log traffic where visibility is useful for troubleshooting or monitoring

## Switching and Wireless Integration

The managed switch provides VLAN tagging and access port assignment using 802.1Q.
Wireless VLAN mapping is handled through the Omada-managed access points. Guest wireless traffic is mapped to the guest VLAN, while trusted wireless traffic remains associated with the trusted LAN.
This allows wired and wireless devices to follow the same segmentation model.

## Lessons Learned

*(Issues I encountered during initial setup and what i leanred from them)*
- VLAN tagging must match between pfSense, the managed switch, and wireless access points
- Trunk and access port roles need to be clearly documented
- PVID settings can break connectivity if misconfigured
- Management access should be preserved before making VLAN changes
- Testing one port or one device at a time reduces recovery risk
