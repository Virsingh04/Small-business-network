# Small Business Network Design (Cisco Packet Tracer)

A segmented network design for a 10-employee small business, built in Cisco Packet Tracer. The network is split into 4 VLANs by department, with inter-VLAN routing configured on a router so departments can communicate in a controlled way.

## Problem

A small business with 10 employees across 4 departments needs a network where:
- Each department's traffic is logically separated (VLANs)
- Departments can still reach each other and the gateway (inter-VLAN routing)
- IP addressing is efficient or a business this size (not wasting large address ranges)

## Network Design
<img width="748" height="509" alt="network-diagram" src="https://github.com/user-attachments/assets/476afde6-7add-477c-9d9f-0c93a7ce004f" />


| Department | VLAN | Subnet | Devices |
|---|---|---|---|
| Management | 10 | 192.168.10.0/28 | PC0, PC1 |
| Sales | 20 | 192.168.20.0/28 | PC2, PC3, PC4, PC5 |
| IT/Admin | 30 | 192.168.30.0/28 | PC6, PC7 |
| HR | 40 | 192.168.40.0/28 | PC8, PC9 |

Each /28 subnet provides 14 usable addresses, enough for each department with room to grow.

## Topology




- **Router0** (Cisco 1941) — router-on-a-stick, one sub-interface per VLAN, acting as the gateway for each department
- **Switch0** (2960-24TT) — assigns each port to its department's VLAN, trunk port carries all VLAN traffic to the router
- **10 PCs** — one static IP per device, matching its department's subnet

## Tools Used

- Cisco Packet Tracer

## Configuration

See `switch-config.txt` and `router-config.txt` for the full command sets used to build this network:
- VLAN creation and naming
- Access port assignment per department
- Trunk port configuration
- Router sub-interfaces with 802.1Q encapsulation (router-on-a-stick)

## Testing & Results

| Test | Result |
|---|---|
| PC1 (Management) → own gateway | 0% packet loss |
| PC1 (Management) → PC3 (Sales), cross-VLAN | 0% loss after initial ARP lookup |
| PC6 (IT/Admin) → own gateway | 0% packet loss |
| PC8 (HR) → own gateway | 0% packet loss |
| PC6 (IT/Admin) → PC8 (HR), cross-VLAN | 0% packet loss |

All 4 VLANs can reach their own gateway and communicate across VLANs through the router.

<img width="695" height="729" alt="ping-test-pc6" src="https://github.com/user-attachments/assets/12b37a35-d499-47e4-8a50-d7757918f8fa" />
<img width="684" height="653" alt="ping-test-pc8" src="https://github.com/user-attachments/assets/55542864-a80e-42cc-a250-4297f360bb5a" />


## A Real Issue I Hit & Fixed

While testing, VLAN 30 (IT/Admin) failed to reach its gateway with 100% packet loss. After checking the router and switch configuration (both correct), I used `show mac address-table` and `show interfaces status` on the switch to trace the problem to the physical layer: PC6 was actually plugged into a switch port that had been assigned to VLAN 40 (HR), not VLAN 30. Reassigning that port to VLAN 30 fixed the issue. This came from checking each layer (IP config → switch port → VLAN assignment) individually rather than guessing.

## What I Learned

- Designing and subnetting a network for a defined number of departments/devices
- VLAN creation, naming, and port assignment on a Cisco switch
- Router-on-a-stick: configuring sub-interfaces with 802.1Q encapsulation for inter-VLAN routing
- Trunk port configuration to carry multiple VLANs over one link
- Diagnosing a real connectivity issue using `show` commands to isolate where in the stack (router vs switch vs physical port) a problem actually lives

## Files

- `router-config.txt` — full router configuration
- `switch-config.txt` — full switch configuration
- `network-diagram.png` — topology diagram
- `ping-test-pc6.png`, `ping-test-pc8.png` — connectivity test screenshots

## Next Steps

- Add DHCP so devices get IPs automatically instead of static configuration
- Add basic ACL/firewall rules between departments
