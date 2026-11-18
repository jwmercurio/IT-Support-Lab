[01-infrastructure.md](https://github.com/user-attachments/files/35045149/01-infrastructure.md)
# Infrastructure and Network Design

**Project:** Enterprise IT Support Lab  
**Scope:** Proxmox virtualization, network segmentation, OPNsense, and baseline connectivity  
**Document status:** Working draft — based on the deployed lab as of November 18, 2026

## Purpose and Design

The lab simulates an enterprise network while remaining separate from the existing home/production services. Proxmox hosts dedicated virtual bridges for Servers, Clients, Management, and DMZ networks. An OPNsense VM connects those networks, enforces firewall policy, and provides a controlled path to the internet through the existing upstream LAN.

This is a **logically isolated lab, not an air-gapped environment**. The internal bridges do not connect directly to physical interfaces; the firewall WAN attaches to the upstream-connected `vmbr0` bridge.

## Architecture

```text
Existing LAN: 192.168.50.0/24 (upstream gateway 192.168.50.1)
                         |
                    vmbr0 — WAN
                         |
                  VM 109: LAB-FW01
                       OPNsense
                         |
          +--------------+--------------+--------------+
          |              |              |              |
        vmbr1          vmbr2          vmbr3          vmbr4
        Servers        Clients        Management     DMZ
     10.20.10.0/24 10.20.20.0/24 10.20.30.0/24 10.20.40.0/24
          |
      VM 110: LAB-DC01
          10.20.10.10
```

## Network and Addressing Inventory

| Bridge | Security zone | Subnet | OPNsense gateway | Function |
|---|---|---|---|---|
| `vmbr0` | Upstream / WAN | `192.168.50.0/24` | Upstream gateway `192.168.50.1` | Internet path and existing LAN |
| `vmbr1` | Servers | `10.20.10.0/24` | `10.20.10.1` | Domain controller and server infrastructure |
| `vmbr2` | Clients | `10.20.20.0/24` | `10.20.20.1` | Planned employee workstations |
| `vmbr3` | Management | `10.20.30.0/24` | `10.20.30.1` | Planned management and administration services |
| `vmbr4` | DMZ | `10.20.40.0/24` | `10.20.40.1` | Planned isolated test services |

| VM | OS / appliance | Connected role | Known address |
|---|---|---|---|
| `109` — `LAB-FW01` | OPNsense | WAN and internal zone gateways | WAN `192.168.50.204/24` |
| `110` — `LAB-DC01` | Windows Server 2025 | AD DS / DNS on Servers network | `10.20.10.10/24` |

**DNS boundary:** `LAB-DC01` uses `10.20.10.10` for domain DNS; planned domain-joined clients should use the AD DNS service rather than arbitrary external resolvers. Required DNS reachability across segments must be tested before client domain joins.

## Key Configuration Decisions

**Dedicated virtual segments.** Multiple internal bridges separate endpoint traffic, domain services, management operations, and exposed test workloads. Keeping internal bridges free of physical uplinks reduces the chance of accidentally placing lab DHCP or DNS services onto the existing LAN.

**Central firewall.** OPNsense is the inter-VLAN/inter-subnet routing and policy point for the lab. Firewall rules should allow only necessary traffic, especially DNS and AD domain-join dependencies between client and server segments. Record *actual rule names, source/destination zones, ports, and test results* once confirmed; do not assume isolation merely because different subnets exist.

**Controlled upstream access.** The lab firewall's WAN interface connects through `vmbr0`. Outbound connectivity has been tested; any access from the lab to the upstream home network should be restricted and explicitly validated.

## Implementation and Verification Record

| Milestone | Current record | Evidence to capture |
|---|---|---|
| Dedicated `IT-Support-Lab` resource pool | Created | Proxmox resource-pool view |
| Internal bridges `vmbr1`–`vmbr4` | Configured | Proxmox node network view |
| OPNsense firewall VM 109 | Deployed | VM hardware/NIC mapping and interface overview |
| WAN connectivity for lab | Reported tested | Outbound DNS lookup / HTTP or package download test |
| Home-network isolation | Reported tested as blocked | Source, destination, rule and blocked-traffic test |
| Domain controller connectivity | VM 110 deployed at `10.20.10.10` | `ipconfig /all`, DNS tests, successful domain-service checks |
| Client-to-domain communication | Not yet tested end-to-end | Test from VM 111 after deployment |
| DMZ policy | Not yet validated | Allow/deny matrix once VM 114 is deployed |

The descriptions above capture the lab's reported progress; screenshots, exported firewall rules, and command outputs should be attached before marking individual controls as **evidence-verified**.

## Security Validation Plan

When the relevant VMs are deployed, perform and record a compact test matrix: client DNS queries to AD DNS; domain join and authentication; server-to-internet update connectivity; denied access from clients to management endpoints; denied access from DMZ to the Servers segment except explicitly required flows; and denied access from lab segments to home/production services. For each test, record the **source, destination, expected result, actual result, and remediation**, if any.

## Troubleshooting and Lessons Learned

The environment initially used a single isolated bridge in the preliminary project plan, with no internet connection. It was redesigned into four internal segments behind a dedicated firewall because the expanded scope required realistic routing, network policy, and end-to-end service testing. This was an architecture change rather than merely an address change.

Record future infrastructure incidents in [`04-testing-and-incidents.md`](04-testing-and-incidents.md) and link them here when relevant, instead of maintaining duplicate troubleshooting narratives in each document.

## Evidence Checklist

- [ ] Proxmox view identifying all five virtual bridges and their uplink assignments
- [ ] VM 109 virtual NIC mapping and OPNsense interfaces
- [ ] OPNsense rule summary, including home-network block policy
- [ ] Connectivity tests showing both permitted and denied outcomes
- [ ] Sanitized network diagram matching the deployed state

*Avoid committing configuration exports containing firewall secrets, administrator credentials, or VPN keys. Lab-only addresses and simulated hostnames may be published when appropriate.*
