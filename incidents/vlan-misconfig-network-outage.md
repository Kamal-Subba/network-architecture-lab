d# Incident: VLAN Guest Network Misconfiguration — Switch Connectivity Outage

**Environment:** Home Lab | pfSense Firewall | Managed Switch | Controller-Managed Wireless Infrastructure

**Duration:** Approximately [1hr]

**Severity:** High

## Impact

* Loss of switch-managed network connectivity
* Loss of access to switch management
* Guest VLAN deployment unsuccessful
* Wireless SSID remained visible but unable to provide network access
* DHCP services unavailable for affected clients

---

## Summary

While deploying a guest VLAN, I applied trunk configuration changes across the switch before validating them on an isolated test port.

The change immediately disrupted communication between the switch and pfSense, preventing VLAN traffic from reaching the firewall. As a result, switch-connected devices lost connectivity and I lost management access to the switch.

---

## Detection

I noticed the issue immediately after applying the VLAN configuration.

### Observed Symptoms

* Switch-connected devices lost connectivity
* Switch management became unreachable
* The guest wireless network remained visible but could not pass traffic
* Clients failed to obtain DHCP leases

---

## Investigation

I started troubleshooting by working through the environment layer by layer.

First, I connected directly to pfSense and confirmed internet access was still available. This allowed me to rule out the firewall itself as the source of the outage.

Next, I reviewed pfSense firewall and DHCP logs to determine whether traffic from the new VLAN was reaching the firewall. I found no evidence of guest VLAN traffic during the incident window.

Because the firewall never saw traffic from the VLAN, I concluded the issue existed between the switch and pfSense, specifically at the VLAN tagging and trunk configuration layer.

---

## Root Cause

The outage was caused by applying an incorrect trunk configuration before validating VLAN behavior on a dedicated test port.

This interrupted communication between the management VLAN and the firewall, preventing VLAN traffic from reaching the routing infrastructure and causing loss of management access.

---

## Remediation

Because remote management access to the switch was unavailable, I began recovery efforts locally.

### Recovery Actions

1. Attempted direct switch access using a manually assigned static IP address.
2. Performed a physical power cycle after management access could not be restored.
3. Reverted VLAN and trunk settings to the previous known-good configuration.
4. Verified restoration of switch management access.
5. Confirmed DHCP functionality and client connectivity.

After reverting the configuration, normal network operation was restored.

---

## Lessons Learned

This incident reinforced several operational practices that I now follow when making infrastructure changes:

* Validate VLAN and trunk changes on a dedicated test port before applying them broadly.
* Confirm management VLAN continuity after every major switch configuration change.
* Maintain a rollback plan before modifying production-connected infrastructure.
* Use out-of-band management access whenever possible.
* Verify end-to-end traffic flow before expanding VLAN deployments to additional devices.

---

## Outcome

I successfully restored network connectivity and later completed the guest VLAN deployment using a staged testing approach.

This incident improved my understanding of VLAN tagging, trunk configuration, management access preservation, and structured troubleshooting methodology within segmented network environments.
