# Lab 01 — Layer 2 Switching, VLANs, RSTP, EtherChannel & ROAS

## Objective
Design and configure a segmented, redundant two-switch corporate LAN featuring multiple VLANs, 802.1Q trunking, LACP link aggregation, Rapid PVST+ Root Bridge optimization, Layer 2 edge protection, and Inter-VLAN routing via Router-on-a-Stick (ROAS).

## Topology
![Lab 01 Topology]("C:\Users\matti\Pictures\Screenshots\topology.png")

## Addressing & VLAN Table

| VLAN ID | Name | Subnet | Default Gateway (`R1`) | Assigned Devices |
| :---: | :--- | :--- | :--- | :--- |
| **10** | `SALES` | `192.168.10.0/24` | `192.168.10.1` (`G0/0.10`) | PC1 (`.10`), PC3 (`.11`) |
| **20** | `IT` | `192.168.20.0/24` | `192.168.20.1` (`G0/0.20`) | PC2 (`.10`), PC4 (`.11`) |
| **99** | `MANAGEMENT` | `192.168.99.0/24` | `192.168.99.1` (`G0/0.99`) | SW1 SVI (`.2`), SW2 SVI (`.3`) |
| **999** | `NATIVE` | *N/A* | `G0/0.999` *(native)* | Dedicated Native VLAN on Trunks |

## Key Technologies Configured
- **VLANs & 802.1Q Trunking:** Segregated user/management traffic, changed Native VLAN to `999`, disabled DTP (`switchport nonegotiate`), and pruned allowed VLANs.
- **EtherChannel (LACP):** Bundled `Fa0/23–24` into `Port-channel 1` (`active` on SW1, `passive` on SW2).
- **Rapid PVST+ & Edge Security:** Forced `SW1` as Root Bridge for VLANs `10, 20, 99` and enabled `PortFast` + `BPDU Guard` on all access ports.
- **Router-on-a-Stick (ROAS):** Configured 802.1Q subinterfaces on `R1 G0/0` for Inter-VLAN routing.

## Verification Commands
- `show vlan brief` — Verified VLAN creation and access port assignments.
- `show etherchannel summary` — Confirmed `Po1(SU)` with LACP active member ports `(P)`.
- `show interfaces trunk` — Verified `Po1` and `Gig0/1` trunking with Native VLAN `999` and allowed VLANs `10,20,99,999`.
- `show spanning-tree` — Confirmed `SW1` elected as Root Bridge running `rstp`.
- **Connectivity Test:** Successful ICMP ping across VLANs (`PC1 -> PC4`) and to switch management SVIs (`PC1 -> 192.168.99.2` and `.3`).

## Troubleshooting Log
- **Issue:** Inter-VLAN communication between PCs worked partially and switches could ping each other on VLAN 99, but `R1` could not reach the switch management SVIs (`192.168.99.2` / `.3`) and `PC1` failed to ping them.
- **Diagnosis:** Running `show interfaces trunk` on `SW1` revealed that only `Po1` was operating as a trunk, while the uplink to `R1` (`Gig0/1`) was missing. Because Cisco routers do not negotiate DTP, the switch port remained in access mode.
- **Resolution:** Manually configured `Gig0/1` on `SW1` as an 802.1Q trunk (`switchport mode trunk`, `switchport nonegotiate`, Native VLAN `999`, allowed VLANs `10,20,99,999`). Full end-to-end connectivity to all SVIs was immediately restored.
