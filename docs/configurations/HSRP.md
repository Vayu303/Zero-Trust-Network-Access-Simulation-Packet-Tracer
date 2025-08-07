# First Hop Redundancy Protocols (FHRP)

We will now enhance the network infrastructure by implementing router redundancy. This is fundamental to eliminate single points of failure in our architecture.

## HSRP Implementation

### Objective

Implement **Hot Standby Router Protocol (HSRP)** between two routers (`R-HQ1` and `R-HQ2`) to provide:

- Layer 3 gateway redundancy
- Automatic failover capability
- Continuous availability for inter-VLAN routing

---

### Devices Involved

| Device   | Role            | IP (VLAN 10 Example) |
| -------- | --------------- | -------------------- |
| R-HQ1    | Active Router   | 192.168.10.254       |
| R-HQ2    | Standby Router  | 192.168.10.253       |
| HSRP VIP | Virtual Gateway | 192.168.10.252       |

> Note: This addressing scheme should be replicated across all routed VLANs (10, 20, 30, 99, 100).

---

### VLAN Gateway Structure

| VLAN | Description | VIP (Gateway)   | R-HQ1 IP        | R-HQ2 IP        |
| ---- | ----------- | --------------- | --------------- | --------------- |
| 10   | IT          | 192.168.10.252  | 192.168.10.254  | 192.168.10.253  |
| 20   | HR          | 192.168.20.252  | 192.168.20.254  | 192.168.20.253  |
| 30   | Sales       | 192.168.30.252  | 192.168.30.254  | 192.168.30.253  |
| 99   | Servers     | 192.168.99.252  | 192.168.99.254  | 192.168.99.253  |
| 100  | Management  | 192.168.100.252 | 192.168.100.254 | 192.168.100.253 |

---

### Physical Connectivity

- Connect **R-HQ2** to the **Core Switch** using a trunk port (e.g., `G0/1`)
- Configure subinterfaces for each VLAN on both routers' trunk interfaces (`G0/0` or `G0/1`), mirroring the existing R-HQ1 configuration

---

### R-HQ2 Subinterface Configuration (Example for VLAN 10)

interface GigabitEthernet0/0.10
encapsulation dot1Q 10
ip address 192.168.10.253 255.255.255.0
standby 10 ip 192.168.10.252
standby 10 priority 90
standby 10 preempt
Note: Use standby group 10 for VLAN 10, 20 for VLAN 20, etc.

R-HQ1 Subinterface Configuration (HSRP Enhancement):
interface GigabitEthernet0/0.10
encapsulation dot1Q 10
ip address 192.168.10.254 255.255.255.0
standby 10 ip 192.168.10.252
standby 10 priority 110
standby 10 preempt

### Key Parameters:

- preempt: Allows higher priority router to reclaim Active role when restored

- priority: Higher value wins (default = 100)

- Repeat this configuration for all participating VLANs.

## DHCP Server Updates (Server-HQ)

For each VLAN pool:

ip dhcp pool VLAN10
network 192.168.10.0 255.255.255.0
default-router 192.168.10.252
dns-server 192.168.99.10
Update all other VLAN pools (20, 30, 99, 100) to use the .252 virtual gateway.

## Verification Tests

Basic Connectivity:

- Ping from PC-IT to .252 (should respond, if not it's probably because ACLs are blocking ICMP)

- Verify inter-VLAN routing with ACLs still functional

Failover Test:

- Disable G0/0 on R-HQ1

- Verify R-HQ2 becomes Active (show standby)

Failback Test:

- Restore R-HQ1

- Verify preemption occurs (if configured)
