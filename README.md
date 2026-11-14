# Troubleshooting #20: Branch Tunnel Down Since the Branch Changed ISP

_Level: Hard — BuildTheNet 3D Sandbox troubleshooting — Tunnels_

**Status: Verified.** Completed on 2026-11-14; every requirement was machine-checked against the network built in BuildTheNet and passed.

Snapshot taken 2026-11-14. Built in a simulation — no physical or production equipment was involved.

## The ticket

**Reported symptom:** Branch-PC can't reach the HQ file server over the site-to-site tunnel since Friday.

WAN Ticket #302: The branch moved to a new ISP circuit on Friday and the provider's installer reconfigured Branch-Gateway for its new public address. Since then the HQ-to-branch GRE tunnel carries nothing and Branch-PC cannot reach HQ-Files. (The tunnel is protected by IPsec in production; this lab models the GRE tunnel and its endpoints, which is where the fault is.)

_Area: Tunnels_

## Required resolution

Restore the site-to-site tunnel so Branch-PC and HQ-Files reach each other through it. Fix it at HQ: Branch-Gateway and the internet are not yours to change.

## Checks

| # | Check | Status | Checked by |
|---|---|---|---|
| 1 | Branch-PC to HQ-Files works | Passed | Sandbox check |
| 2 | HQ-Gateway to Branch-Gateway over the internet still works | Passed | Sandbox check |
| 3 | HQ-Files keeps its documented addressing | Passed | Sandbox check |
| 4 | Branch-PC keeps its documented addressing | Passed | Sandbox check |
| 5 | Internet's configuration is left alone (not yours to change) | Passed | Sandbox check |
| 6 | Branch-Gateway's configuration is left alone (not yours to change) | Passed | Sandbox check |
| 7 | HQ-Gateway GigabitEthernet0/0 keeps its documented addressing | Passed | Sandbox check |
| 8 | HQ-Gateway GigabitEthernet0/1 keeps its documented addressing | Passed | Sandbox check |

## Root cause

HQ-Gateway Tunnel0 still has the branch's old public address (198.51.100.99) as its tunnel destination.

## The fix

What was changed to resolve the ticket, compared with the network as it was reported:

Compared with the starting topology, the learner changed the configuration of 1 device.

**HQ-Gateway**

```diff
  interface Tunnel0
-  tunnel destination 198.51.100.99
+  tunnel destination 198.51.100.2
```

## Result

Every check passed on this snapshot:

> The tunnel points at the branch's new address: Branch-PC reaches HQ-Files through it again.

## Topology

![Topology diagram](topology.svg)

## Addressing

| Device | Interface | Address | Mask | Mode | Gateway |
|---|---|---|---|---|---|
| HQ-Files | FastEthernet0 | 192.168.1.20/24 | 255.255.255.0 | VLAN 1 | 192.168.1.1 |
| HQ-Gateway | GigabitEthernet0/0 | 192.168.1.1/24 | 255.255.255.0 | routed |  |
| HQ-Gateway | GigabitEthernet0/1 | 203.0.113.1/30 | 255.255.255.252 | routed |  |
| HQ-Gateway | Tunnel0 | 172.16.0.1/30 | 255.255.255.252 | routed |  |
| Internet | GigabitEthernet0/0 | 203.0.113.2/30 | 255.255.255.252 | routed |  |
| Internet | GigabitEthernet0/1 | 198.51.100.1/30 | 255.255.255.252 | routed |  |
| Branch-Gateway | GigabitEthernet0/1 | 198.51.100.2/30 | 255.255.255.252 | routed |  |
| Branch-Gateway | GigabitEthernet0/0 | 192.168.2.1/24 | 255.255.255.0 | routed |  |
| Branch-Gateway | Tunnel0 | 172.16.0.2/30 | 255.255.255.252 | routed |  |
| Branch-PC | FastEthernet0 | 192.168.2.10/24 | 255.255.255.0 | VLAN 1 | 192.168.2.1 |

## Device configurations

Running-configs as `show running-config` prints them in the BuildTheNet sandbox. Passwords, secrets, keys and SNMP communities are replaced with `<removed>`.

- [HQ-Gateway](configs/HQ-Gateway.txt)
- [Internet](configs/Internet.txt)
- [Branch-Gateway](configs/Branch-Gateway.txt)

---

Built on [BuildTheNet](https://www.buildthenet.io/sandbox)
