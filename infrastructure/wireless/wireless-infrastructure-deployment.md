# Omada Wireless Infrastructure Deployment

The deployment process also involved several infrastructure and controller adoption issues encountered during onboarding and provisioning.

The troubleshooting process, root cause analysis, and remediation steps are documented separately here:

[Omada AP Adoption Troubleshooting](https://github.com/Kamal-Subba/home-sco-lab/blob/db8acd9bbdd1dc07639a2460f6c2f6a6be4f3d26/wireless/incidents/omada-ap-adoption-troubleshooting.md)

## Overview

This document covers the successful deployment of a controller-managed wireless infrastructure using the TP-Link Omada ecosystem within the home SOC lab environment.


The deployment included:

* Local controller installation
* Managed AP adoption
* Multi-AP wireless deployment
* Wireless mesh configuration
* Centralized SSID management
* VLAN-aware wireless preparation

The goal of this deployment was to transition from standalone wireless management into a centrally managed wireless environment capable of supporting segmented network architecture and future security monitoring workflows.

---

# Environment

## Infrastructure

* pfSense firewall/router
* Managed VLAN-capable switch
* Access points
* management device
* Local controller
* Wireless mesh topology

---

# Objectives

The wireless deployment was designed to achieve the following goals:

* Centralized AP management
* Wireless mesh deployment
* Controller-managed SSID configuration
* VLAN-aware wireless infrastructure
* Scalable multi-AP wireless coverage
* Segmented wireless network support

---

# Controller Deployment

The controller was installed locally on the management device.

## Components

* Software Controller
* Local controller database backend
* Host firewall enforcement

The controller was configured to allow centralized AP provisioning and wireless infrastructure management.

---

# Access Point Deployment

## AP Adoption Workflow

The access points were deployed using a controller-managed onboarding workflow.

### Deployment Process

1. Connected APs to switch ports configured for management VLAN access
2. Allowed APs to obtain DHCP leases from pfSense
3. Verified AP connectivity and management reachability
4. Adopted APs directly through the controller
5. Allowed controller provisioning to complete
6. Verified stable controller-managed state

---

# Infrastructure IP Management

Infrastructure devices were transitioned to DHCP reservation-based management.

## Benefits

* Stable management addressing
* Centralized IP management
* Improved infrastructure consistency
* Simplified troubleshooting and device tracking

---

# Wireless Mesh Deployment

After successful adoption:

* Primary AP remained wired to switching infrastructure
* Secondary AP transitioned into wireless mesh/uplink mode
* Controller automatically managed mesh topology

## Result

* Expanded wireless coverage
* Centralized mesh management
* Seamless wireless  roaming between APs

---

# Wireless Configuration

## Centralized SSID Management

Wireless networks are managed entirely through the controller.

### Current Wireless Deployment

* Controller-managed SSIDs operational
* Multi-AP wireless synchronization functioning
* Centralized wireless authentication management across APs
* Wireless client roaming validated between AP coverage zones

---

# VLAN-Aware Wireless Preparation

The wireless infrastructure was prepared for segmented SSID deployment.

## Current VLAN Usage

| VLAN    | Purpose            |
| ------- | ------------------ |
| VLAN #  | Management Network |
| VLAN #  | Guest Network      |

The environment is prepared for future wireless VLAN tagging and guest network isolation testing.

---

# Security Considerations

The wireless infrastructure was designed with segmentation and centralized management in mind.

Current considerations include:

* Centralized AP management
* VLAN-aware wireless deployment
* Guest network isolation planning
* Infrastructure management separation
* Controlled controller access

Additional hardening and segmentation controls will continue to be implemented as the lab evolves.

---

# Current State

## Operational Components

* Omada controller operational
* Multi-AP deployment operational
* Wireless mesh functioning
* Controller-managed wireless environment stable
* Centralized SSID deployment operational
* Infrastructure management stable

---

# Next Steps

Planned wireless infrastructure improvements include:

* Guest VLAN wireless deployment
* Wireless isolation testing
* Additional AP expansion
* VLAN-aware SSID deployment
* Infrastructure hardening
* Wireless security monitoring integration
* Roaming optimization and tuning

---

# Key Takeaways

This deployment provided hands-on experience with:

* Controller-managed wireless infrastructure
* Multi-AP deployment workflows
* Wireless mesh topology
* VLAN-aware switching
* Infrastructure management practices
* DHCP reservation strategy
* Centralized wireless management

The environment now serves as the wireless foundation for future segmentation, monitoring, and security-focused lab expansion.
