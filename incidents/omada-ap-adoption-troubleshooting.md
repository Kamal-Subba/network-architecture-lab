# Omada AP Adoption Failure — Troubleshooting & Root Cause Analysis

## Summary

What started as what I assumed was a VLAN or switching problem turned into a five-hour multi-layer investigation involving AP adoption failures, an infrastructure IP conflict, firmware behavior that didn't match vendor documentation, and ultimately a host firewall blocking controller provisioning traffic I hadn't accounted for. The environment looked healthy at the network layer the entire time — pings worked, the web UI was reachable, DHCP was handing out leases — but the Omada controller couldn't fully onboard the APs. That gap between "basic connectivity works" and "controller-managed provisioning works" is what made this hard to diagnose.

---

## Environment

| Component | Details |
|---|---|
| Firewall/Router | pfSense |
| Switching | Managed VLAN-capable switch |
| Controller Host | Ubuntu laptop running Omada Software Controller |
| Host Firewall | UFW (active on controller host) |
| Access Points | TP-Link Omada APs |
| Controller Backend | MongoDB |

---

## What I Observed

The APs were clearly alive. They were getting DHCP leases, responding to pings, and reachable through their local web UI. The controller itself was running — MongoDB was up, services were active, and the controller was reachable on TCP 8088 and 8043. But when I watched the controller logs during adoption attempts, what I saw was almost nothing: cloud heartbeat traffic and basic connectivity events. No actual adoption activity. No provisioning handshake.

---

## What I Suspected

My first assumption was a VLAN problem — specifically that management VLAN traffic wasn't flowing correctly between the APs and the controller host. That was the most recent change in the environment, so it was the obvious place to start. When that checked out clean, I moved to pfSense firewall rules, suspecting inter-VLAN restrictions might be silently dropping provisioning traffic. That also came back clean.

At that point I had confirmed the network path was intact, which was frustrating because it meant the problem was somewhere in the controller-to-AP provisioning layer itself — and that's a less predictable space to debug.

---

## What I Tested

### VLAN and Switching

Verified VLAN tagging, PVID assignments, AP uplink port membership, and confirmed the management VLAN was passing traffic correctly. DHCP leases from pfSense were landing on the right scope. This was clean.

### pfSense Firewall Rules

Reviewed inter-VLAN policies, confirmed management network access, and validated that the controller host was reachable from the AP subnet. No blocks found.

### Controller Communication

Confirmed the Omada controller was listening on TCP 8088 and 8043. Validated MongoDB was operational. Reviewed controller logs during live adoption attempts — only heartbeat and connectivity events appeared. No adoption traffic was being processed.

### DHCP Lease Audit

During a detailed review of DHCP leases, I found that one AP had retained a previous static management IP configuration while simultaneously receiving a DHCP lease. This created two competing management identities — the controller couldn't maintain a consistent reference to the device, which explained the intermittent visibility and provisioning instability. I cleared the static config and let DHCP take full control.

### Inform URL Behavior

When manually setting the controller inform URL on the AP, the standard format was rejected by the firmware:

```
http://<controller-ip>:8088/inform
```

The firmware only accepted the bare controller IP. After successful communication, it automatically constructed its own URL, appending dynamic parameters:

```
http://<controller-ip>?dPort=8088&mPort=29814&omadacId=...
```

This behavior wasn't documented in the onboarding guide I was referencing. I validated the actual behavior by watching what the AP reported after a successful partial connection and matched that against what the controller expected.

### Host Firewall

After working through everything above and still not reaching stable adoption, I contacted vender support team. Their first question was whether the host firewall on the controller machine was open for the full Omada port range. I had UFW active and had only opened the two primary ports. Omada's controller uses a range of dynamic ports during the adoption and provisioning handshake — not just 8088 and 8043.

---

## Root Cause

UFW on the Ubuntu controller host was blocking Omada's provisioning and adoption traffic. The two primary ports (8088, 8043) were open, which allowed basic controller visibility and heartbeat communication, but the adoption handshake requires additional ports across the `29801–29817` range for AP discovery, trust establishment, provisioning, and mesh negotiation. Those were all blocked. The result was a controller that could see the APs partially but couldn't complete onboarding — which accounted for the primary adoption and provisioning failures I had observed

The IP conflict was a contributing factor that added noise to the investigation but wasn't the root cause on its own.

---

## What I Changed

Following TP-Link's guidance, I opened the full required port range on UFW, scoped to the management subnet:

```bash
sudo ufw allow from <mgmt-subnet> to any port 8043 proto tcp
sudo ufw allow from <mgmt-subnet> to any port 8088 proto tcp
sudo ufw allow from <mgmt-subnet> to any port 29801:29817 proto tcp
sudo ufw allow from <mgmt-subnet> to any port 29801:29817 proto udp
sudo ufw allow from <mgmt-subnet> to any port 8043 proto udp
sudo ufw allow from <mgmt-subnet> to any port 8088 proto udp
```

> **Note:** TCP on 8088 and 8043 is the primary controller communication path. UDP rules were opened per vendor guidance during onboarding and should be reviewed for reduction once provisioning is stable.

I also resolved the AP static IP conflict by clearing the previous management configuration and allowing DHCP to manage the address exclusively.

---

## Outcome

Immediately after opening the port range, the APs appeared in the controller and completed adoption without issue. Provisioning stabilized, the multi-AP wireless mesh came online, and VLAN-aware wireless was operational across the lab. The full deployment that had been failing for five hours resolved in under two minutes once the actual blocking condition was removed.

---

## Lesson Learned 
*(painfully)*


- Successful network connectivity does not guarantee successful controller provisioning.
- Host-level firewalls should be evaluated alongside network-layer controls during troubleshooting.
- Infrastructure devices should have a single, authoritative management identity before onboarding.
- Vendor documentation should be reviewed before implementing restrictive security controls.

---

## What I Would Do Differently

**Audit the host firewall earlier.** I was thorough on the network path — VLANs, pfSense rules, inter-VLAN routing — but I treated the controller host itself as trusted infrastructure and didn't scrutinize its local firewall until late in the process. In hindsight, any time a service is running on a Linux host and behaving inconsistently, the host firewall is an early candidate, not a last resort.

**Read vendor port documentation before deployment, not during incident recovery.** Omada publishes its full port requirements. I was aware of the two primary ports but hadn't mapped out the full provisioning handshake before standing up the controller.

**Establish stable operation before hardening.** I was applying restrictive UFW rules during initial deployment, which made troubleshooting significantly harder. The better approach is: get the system to a known-good operational state, validate all functionality, then progressively tighten firewall exposure with testing at each step.

**Resolve management identity before provisioning.** Any infrastructure device with a previous static configuration should be fully reset or confirmed clean before bringing it into a new controller environment. The IP conflict added real confusion during diagnosis.
