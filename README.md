# Zero Trust Network Access Simulation – Cisco Packet Tracer

## Overview

This project simulates a **Zero Trust Network Access (ZTNA)** architecture using **Cisco Packet Tracer**. It focuses on **segmentation**, **access control**, and **visibility** using enterprise-grade practices. The implementation includes VLANs, ACLs, AAA with RADIUS, EtherChannel, HSRP, DHCP and DNS Server, etc.
Syslog, SNMP and Honeypot are dummy due to PT limitations.
This is the first version of the project and it contains only one site (HQ), future updates will implement other branches, BGP and OSPF routing, VPNs and more.
This project is meant to improve with future commits in order to create a large corporate network.
Zero Trust Architecture (ZTNA) – Simulation Scope
This project implements Zero Trust Network Access (ZTNA) principles using Cisco ACLs within a fully segmented Packet Tracer topology. While not a full ZTNA stack (no identity provider, posture check, or dynamic access), it successfully simulates a ZTNA-inspired environment based on static access control.

Key Zero Trust features implemented:

🔐 Least Privilege Access: Each VLAN is only allowed to access explicitly authorized services (DNS, HTTP, FTP, Syslog, RADIUS, etc.).

🔒 Micro-Segmentation: Lateral movement between VLANs is blocked unless explicitly permitted.

🚫 Default Deny: All ACLs end with a deny ip any any, enforcing a strict allowlist policy.

📡 Granular Protocol Control: Only required ports and protocols are allowed per service, including DHCP, SSH and HSRP.

> Ideal for students, engineers, and security professionals preparing for real-world deployment or certifications such as CCNA and Security+.

## ✅ Features

- VLAN segmentation: IT, HR, SALES, MGMT, SERVER, RADIUS, BLACKHOLE
- Role-based ZTNA-style extended ACLs (deny-by-default)
- AAA with RADIUS authentication
- SSH access from IT to network devices (MGMT VLAN)
- Syslog centralized logging (dummy but make sure you know the purpose)
- EtherChannel for bandwidth and resilience
- HSRP for high-availability gateway
- DHCP Snooping & Dynamic ARP Inspection (DHCP Rougue, VLAN Hopping and ARP Spoofing prevention)
- Honeypot (i will integrate a simulation for the project, in Packet Tracer there are many limitations)
- Simulated SNMP integration

## 🖥️ Network Topology

| VLAN | Name      | Subnet              | Description                      |
|------|-----------|---------------------|----------------------------------|
| 10   | IT        | 192.168.10.0/24     | Trusted clients                  |
| 20   | HR        | 192.168.20.0/24     | Restricted access users          |
| 30   | SALES     | 192.168.30.0/24     | Internet-only users              |
| 50   | DMZ       | 192.168.50.0/24     | DMZ access                       |
| 99   | SERVER    | 192.168.99.0/24     | Core services (DHCP, DNS, etc.)  |
| 100  | MGMT      | 192.168.100.0/24    | Management subnet                |
| 200  | RADIUS    | 192.168.200.0/24    | AAA server                       |
| 999  | BLACKHOLE | -                   | Disabled/untrusted port VLAN     |
| 254  | ASA-Inside| 192.168.254.0/24    | Network between FW and the LAN   |

## 📁 Repository Structure

```
.
├── network.pkt                # Cisco Packet Tracer project file
├── acls/                      # All extended ACL definitions
│   ├── ASA Firewall ACLs.txt
│   └── Routers ACLs.txt
├── docs/
│   ├── configurations.md         # In-depth documentation
│   ├── implementation.md      # Notes on features and configs
│   └── topology.png           # Network diagram
├── running-config/
│   ├── R-HQ-1.txt
│   ├── R-HQ-2.txt
│   ├── SW-Core.txt
│   ├── SW-IT.txt
│   ├── SW-HR.txt
│   ├── SW-Sales.txt
│   └── FW-HQ.txt
├── README.md                  # This documentation
├── LICENSE                    # License file
└── CHANGELOG.md               # Change history
```

## 🔐 ZTNA Rules Applied

- **Least privilege**: only required access is explicitly allowed
- **Microsegmentation**: each VLAN has its own dedicated ACL
- **No lateral movement**: SALES/HR cannot reach other VLANs
- **Explicit return traffic allowed**
- **All else denied by default**

## 🔧 Key ACL Example (IT-ACL-IN)

```bash
ip access-list extended IT-ACL-IN
 remark DHCP
 permit udp any eq bootpc any eq bootps
 remark DNS
 permit udp any host 192.168.99.10 eq 53
 remark HTTP / HTTPS
 permit tcp any host 192.168.99.12 eq 80
 permit tcp any host 192.168.99.12 eq 443
 remark FTP
 permit tcp any host 192.168.99.11 eq 21
 remark RADIUS
 permit udp any host 192.168.200.10 eq 1812
 permit udp any host 192.168.200.10 eq 1813
 remark SSH to MGMT
 permit tcp any 192.168.100.0 0.0.0.255 eq 22
 remark SYSLOG
 permit udp any host 192.168.99.13 eq 514
 remark Return from servers
 permit ip 192.168.99.0 0.0.0.255 any
 permit ip 192.168.100.0 0.0.0.255 any
 permit ip 192.168.200.0 0.0.0.255 any
 remark Block lateral
 deny ip any 192.168.30.0 0.0.0.255
 deny ip any 192.168.20.0 0.0.0.255
 deny ip any host 192.168.99.16
 deny ip any any
```

## 🚀 Usage Instructions

1. Open `network.pkt` in Cisco Packet Tracer
2. Ensure that interfaces are enabled and switches are configured with VLANs
3. Check ACLs on routed subinterfaces (e.g., `int g0/0.10`), try to write new ones on your own and test
4. Use PC in VLAN 10 (IT) to:
    - SSH into switches (only SW-IT and SW-HR at the moment)
5. Use PC in VLAN 20 or 30 to:
    - Obtain IP via DHCP
    - Resolve DNS
    - Access HTTP, FTP, HTTPS
6. Test lateral movement blocks from SALES/HR to IT

## 🧪 Testing Scenarios

- ✅ DHCP from VLAN 20 and 30
- ✅ ACL restrictions between departments
- ✅ RADIUS login authentication (local server)
- ✅ SSH from IT → MGMT
- ✅ HSRP failover
- ❌ SNMP and Meraki are simulated only

## 🧱 Future Plans

- Full-dual stack IPv4/IPv6
- Adding new sites, implementing dynamic routing, firewalls, endpoint protection, APs, VPN and more
- YAML-based ZTNA policy abstraction
- Ansible/CLI script generation
- Simulation of alerts from honeypot
- ZTNA rules export to firewall formats
- Extended SNMP traps and visual dashboards
- VMs and Containers for improving the quality and complexity of the simulation
- New branch site, with a full Layer 3 network

## Security Plans

- I would like ti implement a security section, where we can discuss more about best security practices, vulnerabilities and protections


## 📜 License

MIT License - see the [LICENSE](./LICENSE) file for more details.

## 🤝 Contributing

Feel free to fork, open issues, or suggest improvements.  
This project is intended to evolve and demonstrate best practices.
