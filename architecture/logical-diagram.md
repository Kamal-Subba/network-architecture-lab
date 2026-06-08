# Logical Network Diagram


## Purpose

This document is written to provide a higher level logical view of the network architecture lab than just the diagram. 

The diagram shows how the firewall, managed switch, wireless infrastructure, monitoring server, endpoints, and future segmented networks are connected at a design level.

This doc will not include IP addresses, physical cabiling details or sensitive config values by intent.

---

## Logical Diagram

                         ┌──────────────┐
                         │   Internet   │
                         └──────┬───────┘
                                │
                         ┌──────▼───────┐
                         │  ISP Modem   │
                         └──────┬───────┘
                                │
                         ┌──────▼───────┐
                         │   pfSense    │
                         │ Firewall/RT  │
                         └──────┬───────┘
                                │
                         ┌──────▼───────┐
                         │ Managed      │
                         │ Switch       │
                         └──────┬───────┘
                                │
        ┌───────────────────────┼────────────────────────┐
        │                       │                        │
┌───────▼────────┐      ┌───────▼────────┐       ┌───────▼────────┐
│ Management     │      │ Wireless APs   │       │ Wazuh Server   │
│ Workstation    │      │ Omada Managed  │       │ Ubuntu Server  │
└────────────────┘      └───────┬────────┘       └───────┬────────┘
                                │                        │
              ┌─────────────────┼──────────────┐         │
              │                 │              │         │
      ┌───────▼───────┐ ┌───────▼───────┐      │         │
      │ Trusted SSID  │ │ Guest SSID    │      │         │
      │ LAN Access    │ │ VLAN Isolated │      │         │
      └───────────────┘ └───────────────┘      │         │
                                               │         │
        ┌──────────────────────────────────────▼─────────▼──────┐
        │                  Monitoring / Logging                 │
        │        pfSense logs + endpoint agent telemetry        │
        └───────────────────────────────────────────────────────┘

---

## Design Summary

The lab is built around pfSense as the primary firewall and routing point. Network segmentation is handled through VLANs, with pfSense enforcing inter-VLAN firewall rules.

The managed switch provides VLAN-aware connectivity between wired devices, wireless access points, and lab infrastructure. Wireless segmentation is handled through Omada-managed access points, where SSIDs can be mapped to specific VLANs.

Wazuh provides centralized monitoring and log visibility for supported systems in the lab.

---

## Traffic Flow Summary

At a high level, traffic is routed through pfSense, with VLAN segmentation enforced by firewall policy. Wireless clients are assigned to the appropriate network through SSID-to-VLAN mapping, and infrastructure/endpoint telemetry is forwarded to monitoring systems via push-based logging.

--

## Major Components

| Component | Role |
|---|---|
| pfSense | Firewall, routing, VLAN interfaces, DHCP, and inter-VLAN policy enforcement |
| Managed Switch | VLAN tagging, access ports, trunk ports, and wired device connectivity |
| Omada Access Points | Wireless access, SSID management, and wireless VLAN mapping |
| Wazuh Server | Centralized logging, endpoint telemetry, and monitoring |
| Management Workstation | Administrative system used to manage lab infrastructure |
| Lab Endpoints | Systems used for testing, monitoring, and future projects |

---

## Traffic Flow Overview

At a high level:

- Internet traffic passes through pfSense
- VLAN routing is controlled by pfSense firewall rules
- Wired VLAN access is handled by the managed switch
- Wireless VLAN access is handled by Omada SSID-to-VLAN mapping
- Logs and endpoint telemetry are forwarded to Wazuh where supported
- Management access is intended to be limited to trusted administrative systems

---

## Sanitization Notes

This diagram intentionally excludes:

- Real IP addresses
- MAC addresses
- Public IP information
- Device serial numbers
- Exact hostnames
- Credentials or management URLs
