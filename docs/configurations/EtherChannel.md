# EtherChannel Implementation

DISCLAIMER

With the actual version of Packet Tracer, it is not possibile to configure EtherChannel with DHCP Server mechanism. It is not possibile to write the command `ip dhcp snooping trust` on the **Port-channel 1** interface. For this very reason, I will put only ONE EtherChannel link for demonstration, setting up static IP addresses for VLAN 10. Then I will leave the other VLAN as they are, in order to demonstrate that DHCP Server is working properly.

With real equipment it is enough to configure DHCP snooping only on the physical ports used for the EtherChannel connection but in Packet Tracer no. Even if done, DHCP messages will not be able to flow through the EtherChannel link.

## Overview

To improve bandwidth, resilience, and reduce STP blocking, we implement **LACP-based EtherChannel** between the **Core Switch** and each **Access Switch** (e.g., `SW-IT`, `SW-HR`, `SW-Sales`). This allows multiple physical links to act as a single logical connection (Port-channel), combining speed and failover in one structure.

---

## Benefits of EtherChannel

| Feature                     | Benefit                                                          |
| --------------------------- | ---------------------------------------------------------------- |
| **Link Redundancy**         | One or more physical links can fail without losing connectivity. |
| **Bandwidth Aggregation**   | Multiple links operate as one; e.g., 2x1Gbps = 2Gbps channel.    |
| **Simplified STP Behavior** | STP treats the EtherChannel as a single link.                    |
| **Simpler Management**      | All interfaces are configured and managed as one Port-channel.   |

---

## EtherChannel Configuration (LACP - IEEE 802.3ad)

> Configuration is based on using 2 FastEthernet links (e.g., Fa0/1 and Fa0/11).

---

### Core Switch (SW-Core)

```bash
interface range FastEthernet0/1 - 11
 channel-group 1 mode active
exit

interface Port-channel1
 description EtherChannel to SW-IT
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,99,100
```

---

### Access Switch (SW-IT)

```bash
interface range FastEthernet0/1 - 11
 channel-group 1 mode active
exit

interface Port-channel1
 description EtherChannel to SW-Core
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,99,100
 ip dhcp snooping trust   -> this is not possibile in Packet Tracer
 ip arp inspection trust  -> this is not possibile in Packet Tracer
```

---

## Notes

- **LACP (mode active)** negotiates the EtherChannel using open standards.
- The same `channel-group` number must be used on both ends (here: `1`).
- Only interfaces with the **same speed and duplex** can form EtherChannel.
- Avoid applying **port-security** on trunk interfaces or Port-channels.
- Ensure **VLANs match** on both sides.
- Verify with:

```bash
show etherchannel summary
```

---

## Spanning Tree Protocol

Once EtherChannel is established:

- STP sees `Port-channel1` as a single logical link.
- No ports are blocked by STP between Core and Access switches.

---

## Common Issues

| Symptom                   | Possible Cause                                             |
| ------------------------- | ---------------------------------------------------------- |
| Interfaces remain down    | Speed/duplex mismatch, channel-group misalignment          |
| Port not added to channel | One side using `on`, the other using `active` or `passive` |
| STP blocking              | Trunk mismatch or BPDU issues                              |

---

## Repeat for All Access Switches

Repeat this configuration with `SW-HR`, `SW-Sales`, or other future access switches by:

- Choosing a unique `Port-channel` ID for each new pair (e.g., `2`, `3`, etc.).
- Matching allowed VLANs and mode.
