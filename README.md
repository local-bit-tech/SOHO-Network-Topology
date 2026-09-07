# SOHO Network Topology

A small office / home office network built in Cisco Packet Tracer, configured from
bare devices to a segmented, dynamically routed, and Layer 2–hardened network.

The build covers VLAN segmentation, Spanning Tree root placement, LACP EtherChannel,
Router-on-a-Stick inter-VLAN routing, OSPF, centralized DHCP with a relay agent, and
DHCP Snooping with Dynamic ARP Inspection.

Every step is documented with the exact CLI, the reasoning behind it, and verification output.

---

## Topology

**Before configuration**

<img width="1217" alt="Topology before configuration" src="https://github.com/user-attachments/assets/2e981216-f37f-4603-8296-9f65452b8e41" />

**After configuration**

<img width="1289" alt="Topology after configuration" src="https://github.com/user-attachments/assets/0a13105d-16a0-444f-b8f6-0be69dbc1549" />

---

## Walkthrough

| # | Guide | Covers |
|---|-------|--------|
| 1 | [VLANs and STP](Steps/1.%20VLAN's%20and%20STP.md) | VLAN creation, access ports, STP root primary/secondary, PortFast, trunking |
| 2 | [EtherChannel and ROAS](Steps/2.%20Etherchannel%20and%20ROAS.md) | LACP link aggregation, sub-interfaces, 802.1Q encapsulation |
| 3 | [OSPF and DHCP](Steps/3.%20OSPF%20and%20DHCP.md) | OSPF area 0 (two config methods), DHCP server + relay, intra/inter-VLAN path testing |
| 4 | [DHCP Snooping and ARP Inspection](Steps/4.%20DHCP%20Snooping%20and%20ARP%20Inspection.md) | Trusted/untrusted ports, snooping binding table, DAI |

---

## Addressing

| VLAN | Name | Subnet | Gateway | Access ports |
|------|------|--------|---------|--------------|
| 10 | HR | 192.168.1.0/24 | 192.168.1.254 (R2 g0/0/1.1) | SW1 f0/3, SW2 f0/3 |
| 20 | IT | 192.168.2.0/24 | 192.168.2.254 (R2 g0/0/1.2) | SW1 f0/4, SW2 f0/4 |

## Device roles

| Device | Role |
|--------|------|
| **R1** | OSPF (configured per-interface), DHCP server for both pools |
| **R2** | OSPF (configured with `network` statements), ROAS gateway, DHCP relay via `ip helper-address` |
| **SW1** | STP root primary for VLAN 10 / secondary for VLAN 20, trunk to R2, P1 to SW2 |
| **SW2** | STP root primary for VLAN 20 / secondary for VLAN 10, P1 to SW1 |

Interswitch links f0/1–2 are bundled into **Port-channel1** using LACP (802.3ad).

---

## Running the lab

Requires **Cisco Packet Tracer 8.x** (free with a Cisco Networking Academy account).

| File | Description |
|------|-------------|
| [`packets/1. Pre-configuration.pkt`](packets/1.%20Pre-configuration.pkt) | Cabled topology with no configuration — start here to follow along |
| [`packets/2. Post-configuration.pkt`](packets/2.%20Post-configuration.pkt) | Completed build, for reference or verification |

Open the pre-configuration file and work through the guides in order. Each one builds on
the last, so skipping ahead will leave you without prerequisites (ROAS won't work without
the trunk from step 1, DHCP won't work without OSPF from step 3).

---

## Verifying the build

| Check | Command |
|-------|---------|
| VLAN membership | `show vlan brief` |
| STP root per VLAN | `show spanning-tree vlan 10` |
| Trunk allowed VLANs | `show interfaces trunk` |
| EtherChannel status | `show etherchannel summary` |
| OSPF neighbors / routes | `show ip ospf neighbor`, `show ip route ospf` |
| DHCP leases | `ipconfig /renew` on a PC |
| Snooping bindings | `show ip dhcp snooping binding` |
| DAI status | `show ip arp inspection` |

Inter-VLAN traffic should traverse R2; intra-VLAN traffic should stay on the switches.
`tracert` between hosts confirms both paths.

---

## Packet Tracer caveats

- `ip dhcp snooping trust` is not supported on `port-channel1` in Packet Tracer. Workaround:
  disable the LAG with `no channel-group 1 mode active` on f0/1–2, then re-apply
  `switchport trunk allowed vlan 10,20` — tearing down the bundle clears the allowed VLAN list.
- `no ip dhcp snooping information option` is required, otherwise R2 drops relayed
  DHCP Discover messages carrying Option 82.

---

## Skills demonstrated

Layer 2 segmentation, Spanning Tree design and load balancing, Link aggregation,
Inter-VLAN routing, OSPF, Centralized DHCP with relay,
Layer 2 attack mitigation (rogue DHCP, ARP spoofing), CLI verification and troubleshooting

---

## Notes

This is a lab environment built for learning. Production deployments would typically use a
dedicated DHCP server rather than a router, and would add port security, AAA, SSH management
access, and logging.

## License

MIT
