# Single-Area OSPF Lab

## Objective
Configure and verify a basic single-area OSPF network connecting two remote LANs through a central router, demonstrating fundamental OSPF neighbor adjacency, route propagation, and end-to-end connectivity.

## Topology
![Network Topology](topology/network-diagram.png)

| Router  | Role            | Area   |
|---------|-----------------|--------|
| Router0 | Central/hub router | Area 0 |
| Router1 | Edge router (LAN A) | Area 0 |
| Router2 | Edge router (LAN B) | Area 0 |

### Point-to-point links
| Link               | Subnet         | Area   |
|--------------------|----------------|--------|
| Router0 – Router1  | 10.0.0.0/30    | Area 0 |
| Router0 – Router2  | 20.0.0.0/30    | Area 0 |

### LAN segments
| Segment        | Subnet             | Hosts        | Connected via |
|----------------|---------------------|--------------|---------------|
| Switch0 LAN    | 192.168.10.0/24     | PC0, PC1     | Router1       |
| Switch1 LAN    | 192.168.20.0/24     | PC2, PC3     | Router2       |

## Design Notes
- All routers reside in a single OSPF area (Area 0), which is the correct minimum design for a small topology — multi-area design only becomes necessary as the network scales and LSA flooding domains need to be contained.
- Router0 acts purely as a transit router between the two edge networks; it advertises no LAN segments of its own.
- Point-to-point /30 links between routers avoid DR/BDR election overhead, since OSPF treats point-to-point network types differently from broadcast (Ethernet) segments.

## Configuration
Full running-configs for each device are in [`/configs`](configs/).

Example OSPF configuration pattern (Router0):
```
router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
 network 20.0.0.0 0.0.0.3 area 0
```

## Verification
Screenshots in [`/screenshots`](screenshots/) confirm:
- `show ip ospf neighbor` — FULL adjacency established between Router0–Router1 and Router0–Router2
- `show ip route ospf` — Router1 learns the 192.168.20.0/24 route via Router0, and vice versa
- End-to-end ping/traceroute from PC0 (192.168.10.0/24) to PC2 (192.168.20.0/24), confirming traffic transits Router0 correctly

## Security Considerations
- No OSPF authentication is configured. In production, MD5 or SHA authentication on OSPF interfaces prevents an unauthorized device from forming a rogue adjacency and injecting false routes.
- Passive-interface should be applied on Router1's and Router2's LAN-facing interfaces (toward Switch0/Switch1) so OSPF hellos are never sent toward end-user segments — this reduces the attack surface for neighbor spoofing or LSA injection from a compromised host.
- Since this is a flat single-area design, a compromised or misconfigured router anywhere in the topology can affect the entire OSPF domain — this is one practical reason larger networks move to multi-area design (see companion multi-area OSPF lab).

## Files
```
.
├── README.md
├── topology/
│   └── network-diagram.png
├── configs/
│   ├── router0-config.txt
│   ├── router1-config.txt
│   └── router2-config.txt
├── screenshots/
│   ├── ospf-neighbors.png
│   ├── ip-route-ospf.png
│   └── ping-traceroute-verification.png
└── packet-tracer/
    └── single-area-ospf-lab.pkt
```
