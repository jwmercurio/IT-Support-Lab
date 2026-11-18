[README.md](https://github.com/user-attachments/files/35045055/README.md)
# Enterprise IT Support Lab

Hands-on enterprise IT support and systems administration lab built with Proxmox VE, OPNsense, Windows Server 2025, Active Directory, and segmented virtual networks.

> **Project status — November 18, 2026:** Network infrastructure and the initial Active Directory deployment are operational. Workstation integration, Group Policy, help desk workflows, monitoring, and additional testing are in progress or planned.

## Overview

This project simulates a small company's IT environment to practice the tasks performed by Help Desk Technicians, Desktop Support Specialists, and Junior Systems Administrators. It emphasizes deploying infrastructure, managing users and endpoints, enforcing access controls, solving incidents, and recording reproducible technical evidence.

The environment runs on a personal Proxmox server. Four dedicated internal network segments are connected through an OPNsense firewall VM, which also provides controlled internet access through an existing upstream LAN. The lab is **segmented, not physically air-gapped**: internal bridges have no physical uplinks, while the firewall's WAN interface connects to the upstream network.

## Objectives

- Administer Windows Server, Active Directory Domain Services (AD DS), DNS, and Group Policy.
- Create employee accounts, department-based organizational units, and role-based security groups.
- Practice employee onboarding, offboarding, permissions, and delegated help desk administration.
- Deploy and domain-join a Windows 11 employee workstation.
- Operate a GLPI ticketing and knowledge-base system.
- Use PowerShell for repeatable administration and reporting.
- Configure monitoring, troubleshoot simulated incidents, and document verification results.
- Practice routing, firewall policy, network segmentation, and safe testing in a controlled environment.

## Technology Stack

| Technology | Role |
|---|---|
| Proxmox VE 9.2 | Virtualization and virtual networking |
| OPNsense | Routing, firewall policies, and internet access for lab networks |
| Windows Server 2025 | Domain controller, AD DS, DNS, and planned Group Policy |
| Windows 11 Pro | Planned domain-joined employee workstation |
| Debian 13 | Planned Linux service hosts |
| GLPI | Planned IT service desk and asset/ticket management |
| Zabbix | Proposed infrastructure monitoring |
| PowerShell | Planned administrative automation |

## Network Architecture

```text
Upstream / home LAN: 192.168.50.0/24
               |
          vmbr0 (WAN)
               |
    VM 109: LAB-FW01 (OPNsense)
      WAN: 192.168.50.204/24
               |
       +-------+-------+-------+-------+
       |               |       |       |
     vmbr1           vmbr2   vmbr3   vmbr4
     Servers         Clients Management DMZ
 10.20.10.0/24  10.20.20.0/24  10.20.30.0/24  10.20.40.0/24
       |
   VM 110: LAB-DC01
    10.20.10.10
```

| Bridge | Segment | Subnet | OPNsense gateway | Status |
|---|---|---|---|---|
| `vmbr0` | Existing upstream network / firewall WAN | `192.168.50.0/24` | Upstream `192.168.50.1` | Connected |
| `vmbr1` | Servers | `10.20.10.0/24` | `10.20.10.1` | Configured |
| `vmbr2` | Employee clients | `10.20.20.0/24` | `10.20.20.1` | Configured; client deployment pending |
| `vmbr3` | Management | `10.20.30.0/24` | `10.20.30.1` | Configured; service deployment pending |
| `vmbr4` | DMZ | `10.20.40.0/24` | `10.20.40.1` | Configured; workload deployment pending |

**Security approach:** OPNsense is the intended routing and policy-enforcement point between lab segments. Internet access from the lab has been tested, and access to the existing production/home network was tested as blocked. Future testing will document per-segment rules and verify that the intended restrictions continue to hold.

## Virtual Machine Inventory

| VM ID | Hostname | Operating system | Purpose | Status |
|---|---|---|---|---|
| 109 | `LAB-FW01` | OPNsense | Firewall, internal gateways, WAN connectivity | Deployed |
| 110 | `LAB-DC01` | Windows Server 2025 | AD DS, DNS, identity administration | Deployed |
| 111 | `LAB-CLIENT01` | Windows 11 Pro | Employee workstation, domain join, GPO testing | Planned |
| 112 | `LAB-HELPDESK01` | Debian 13 | GLPI ticketing and IT knowledge base | Planned |
| 113 | `LAB-MON01` | Debian / Zabbix | Monitoring and alerting | Proposed |
| 114 | `LAB-WEB01` | Linux | Segmented DMZ test service | Proposed |

## Active Directory Environment

| Setting | Value |
|---|---|
| Domain | `corp.itlab.test` |
| NetBIOS name | `CORP` |
| Domain controller | `LAB-DC01` (VM 110) |
| IP address | `10.20.10.10/24` |
| Default gateway | `10.20.10.1` |
| DNS server | `10.20.10.10` |
| Services | AD DS and DNS |

### Organizational Unit Structure

```text
corp.itlab.test
├── LAB
│   ├── Users
│   │   ├── IT
│   │   ├── HR
│   │   ├── Finance
│   │   └── Sales
│   ├── Workstations
│   ├── Servers
│   ├── Groups
│   ├── Admins
│   └── Service Accounts
└── Domain Controllers (default container for LAB-DC01)
```

### Security Groups

- `GG_IT_Staff`
- `GG_HR_Staff`
- `GG_Finance_Staff`
- `GG_Sales_Staff`
- `GG_Helpdesk_Tier1`
- `GG_Helpdesk_Tier2`

Eight simulated employee accounts have been created and assigned security groups. The IT and Sales users' OU placement needs review because those accounts were placed in the opposite departmental OUs during setup. This is tracked as a configuration cleanup item; the group assignments were created separately.

### Verification Completed

- Installed Windows Server 2025, VirtIO drivers, and QEMU Guest Agent.
- Assigned the domain controller a static address and DNS configuration.
- Installed AD DS and DNS and promoted `LAB-DC01` into `corp.itlab.test`.
- Verified the domain controller role and that core services (`NTDS`, `DNS`, `Netlogon`) were running.
- Created the OU hierarchy, employee accounts, and six security groups.

**Not yet completed:** Group Policy configuration, delegation of help desk password-reset rights, Windows 11 domain join, and end-to-end onboarding/offboarding validation.

## Implementation Roadmap

### Phase 1 — Virtualization and Networking

- [x] Create the `IT-Support-Lab` resource pool.
- [x] Configure internal bridges `vmbr1` through `vmbr4`.
- [x] Deploy OPNsense as VM 109 and configure segment gateways.
- [x] Connect the firewall WAN to `vmbr0` for controlled internet access.
- [x] Test internet connectivity and blocked access to the home/production network.
- [ ] Record firewall rule evidence and detailed network test results in GitHub.

### Phase 2 — Windows Server and Active Directory

- [x] Deploy Windows Server 2025 as VM 110 (`LAB-DC01`).
- [x] Configure the static IP, gateway, and DNS settings.
- [x] Install AD DS and DNS and create `corp.itlab.test`.
- [x] Create department OUs and administrative/service account OUs.
- [x] Create eight simulated employees and six security groups.
- [ ] Review/correct IT and Sales users' OU placement.
- [ ] Configure and test help desk delegation (for example, password resets).
- [ ] Create and verify baseline Group Policy Objects (GPOs).
- [ ] Validate employee onboarding and offboarding procedures.

### Phase 3 — Employee Workstation Support

- [ ] Deploy Windows 11 Pro as VM 111 (`LAB-CLIENT01`) on `vmbr2`.
- [ ] Confirm DNS and network access to the domain controller.
- [ ] Join the workstation to `corp.itlab.test`.
- [ ] Test domain logins, user profiles, permissions, and GPO application.
- [ ] Document common login, DNS, and connectivity issues.

### Phase 4 — Service Desk Operations

- [ ] Deploy Debian and GLPI as VM 112 (`LAB-HELPDESK01`).
- [ ] Create simulated tickets for account, workstation, network, and access incidents.
- [ ] Document troubleshooting, escalation, resolution, and ticket closure.
- [ ] Develop an internal support knowledge base.

### Phase 5 — Automation and Monitoring

- [ ] Create PowerShell scripts for account provisioning, auditing, and reporting.
- [ ] Deploy monitoring (Zabbix proposed) and configure useful alerts.
- [ ] Simulate failures and record the detection, diagnosis, and resolution process.

### Phase 6 — DMZ and Security Validation

- [ ] Deploy an isolated test web server in the DMZ.
- [ ] Define and verify cross-segment firewall policies.
- [ ] Record permitted/denied connectivity tests and remediation steps.

## Documentation

Documentation is organized around **four technical narratives**, not an installation guide for every virtual machine. The README provides the high-level overview; supporting documents show important configuration decisions, validation, and real troubleshooting evidence.

```text
IT-Support-Lab/
├── README.md
├── docs/
│   ├── 01-infrastructure.md
│   ├── 02-active-directory.md
│   ├── 03-support-operations.md
│   └── 04-testing-and-incidents.md
├── evidence/         # Selected screenshots, diagrams, and redacted test results
└── scripts/          # Working administration scripts as developed
```

| Document | Scope | State |
|---|---|---|
| [Infrastructure](docs/01-infrastructure.md) | Proxmox, bridges, subnets, OPNsense, connectivity, isolation | Draft in progress |
| Active Directory | Windows Server, domain, DNS, OU and group design, GPOs, delegation | Planned write-up; deployment underway |
| Support Operations | Workstations, employee lifecycle, GLPI, service desk procedures | Planned |
| Testing and Incidents | Evidence-led troubleshooting, failed tests, fixes, and retesting | Planned; update throughout the lab |

Each write-up records **what was built, why it was configured that way, how it was validated, what went wrong, and what we learned**. Routine installation clicks are omitted. Evidence and scripts are linked from the corresponding write-ups only when they add value.

### Immediate Next Steps

1. Replace the outdated README with the current project overview.
2. Document the deployed Proxmox/OPNsense environment in `docs/01-infrastructure.md` and add verification evidence.
3. Document the existing Active Directory configuration in `docs/02-active-directory.md`, distinguishing verified settings from planned GPO and delegation work.
4. Continue VM 110 configuration and record only meaningful design changes, tests, and troubleshooting outcomes.

*All published screenshots, configurations, and outputs will be reviewed for credentials and other sensitive information.*

---

*This is a learning and portfolio environment using simulated users and services. Completion marks reflect work performed as of November 18, 2026, not features merely planned for a future phase.*
