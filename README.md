# Enterprise Windows Server 2022 — Active Directory, GPO, DNS & DHCP Lab

A complete **Windows Server 2022 Domain Controller (`corp.local`)** home lab built on **VMware Workstation Pro**, demonstrating enterprise identity management, Organizational Unit (OU) design, AD-integrated DNS, authorized DHCP scope provisioning, and Group Policy (GPO) security logon auditing.

> **Why this project matters:** Enterprise networks rely on Active Directory, DNS, DHCP, and Group Policy as the core identity and access backbone. This lab demonstrates hands-on deployment via both **PowerShell** and **Windows Server Manager GUI**, backed by live security telemetry verification in **Windows Event Viewer (`Event ID 4624`)**.

---

## What this lab implements

| Feature | Implementation |
| --- | --- |
| **Hypervisor** | VMware Workstation Pro (`Windows Server 2022 Standard — Desktop Experience`, 4 GB RAM, 60 GB NVMe) |
| **Domain Controller** | `lab-2026.corp.local` (`192.168.248.128/24`, Gateway `192.168.248.2`) — Primary Domain Controller (PDC) |
| **Active Directory (AD DS)** | Custom Organizational Units (`IT-Operations`, `Workstations`), Security Group (`Network-Admins`), Domain User (`Arpan Admin`) |
| **DNS Server** | AD-integrated Forward Lookup Zones (`corp.local`, `_msdcs.corp.local`) with verified `SOA`, `NS`, and static `Host (A)` records |
| **DHCP Server** | AD-authorized DHCP server with active IPv4 scope `Corp-LAN-Scope` (`192.168.248.150 – 192.168.248.200/24`) |
| **Group Policy (GPO)** | Domain-linked GPO `Security-Audit-Policy` enforcing `Audit logon events` and `Audit account logon events` (`Success, Failure`) |
| **Security Telemetry** | Verified real-time security logs in `Event Viewer -> Security` (`Event ID 4624` Logon, `4672` Special Logon, `4768/4769` Kerberos) |

---

## Skills demonstrated

- VMware Workstation Pro virtual machine provisioning and snapshot management
- Windows Server 2022 installation, static IPv4 addressing, and role deployment via PowerShell (`Install-WindowsFeature`, `Install-ADDSForest`) and Server Manager
- Active Directory Domain Services (`dsa.msc`): forest/domain promotion, Organizational Units (OUs), security groups, and least-privilege user administration
- DNS Manager (`dnsmgmt.msc`): forward lookup zone verification, `SOA`, `NS`, and `Host (A)` record management
- DHCP Administration (`dhcpmgmt.msc`): Active Directory authorization, IPv4 scope creation, address pool allocation, and router/DNS scope options
- Group Policy Management (`gpmc.msc`): creating, linking, and enforcing security audit policies (`gpupdate /force`)
- Windows Security Event Log analysis (`eventvwr.msc`): verifying authentication and logon telemetry (`Event ID 4624`, `4672`, `4768`, `4769`)

---

## Lab Architecture

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                    VMware Workstation Pro (192.168.248.0/24)            │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │  Primary Domain Controller: lab-2026.corp.local                 │   │
│   │  IPv4: 192.168.248.128/24  |  Gateway: 192.168.248.2            │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │  • Active Directory (AD DS): corp.local                         │   │
│   │      ├── OU: IT-Operations (User: Arpan Admin, Grp: Net-Admins) │   │
│   │      └── OU: Workstations                                       │   │
│   │  • DNS Server: AD-Integrated Zone (corp.local, SOA, NS, Host A) │   │
│   │  • DHCP Server: Scope [192.168.248.0] Corp-LAN-Scope (Active)   │   │
│   │  • Group Policy: Security-Audit-Policy (Logon Success/Failure)  │   │
│   │  • Event Viewer: Security Log Telemetry (Event IDs 4624, 4672)  │   │
│   └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Verification Evidence & Screenshots

### 1. Server Manager — AD DS & DNS Roles Active
Installed `AD-Domain-Services` and `DNS` roles with management tools and promoted the server to a Domain Controller for `corp.local`.
![Server Manager AD DS and DNS Dashboard](screenshots/02-server-manager-adds-dns-dashboard.png)

### 2. Active Directory Users, Groups & Organizational Units (`dsa.msc`)
Created Organizational Units (`IT-Operations` and `Workstations`), provisioned user account `Arpan Admin`, and assigned membership to the `Network-Admins` security group.
![Active Directory OUs and Users](screenshots/01-active-directory-ous-and-users.png)

### 3. DNS Manager — Forward Lookup Zone Verification (`dnsmgmt.msc`)
Verified AD-integrated DNS records (`SOA`, `NS`, and static `Host (A)` records resolving `lab-2026.corp.local` to `192.168.248.128`).
![DNS Manager Forward Lookup Zone](screenshots/05-dns-manager-forward-lookup-zone.png)

### 4. DHCP Server — Active IPv4 Scope (`dhcpmgmt.msc`)
Authorized `lab-2026.corp.local` in Active Directory and activated IPv4 scope `Corp-LAN-Scope` (`192.168.248.150 – 192.168.248.200/24`).
![DHCP Active Scope](screenshots/03-dhcp-server-active-scope.png)

### 5. Group Policy Management — Security Logon Auditing (`gpmc.msc`)
Created and linked GPO `Security-Audit-Policy` across `corp.local`, enabling **Success & Failure** auditing for both Account Logon Events and Logon Events.
![Group Policy Security Auditing](screenshots/04-group-policy-security-auditing.png)

### 6. Windows Event Viewer — Security Audit Log Verification (`eventvwr.msc`)
Verified real-time security telemetry generation in `Windows Logs -> Security`, confirming **Event ID 4624** (*An account was successfully logged on*), **Event ID 4672**, and **Kerberos authentication events (4768/4769)** on `Lab-2026.corp.local`.
![Event Viewer Security Audit 4624](screenshots/06-event-viewer-security-audit-4624.png)

---

## Key PowerShell & CLI Commands Used

```powershell
# 1. Verify Network Configuration
ipconfig /all

# 2. Install Active Directory Domain Services & DNS Server Roles
Install-WindowsFeature -Name AD-Domain-Services, DNS -IncludeManagementTools

# 3. Configure DNS Client Addressing
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses ("127.0.0.1","192.168.248.2")

# 4. Promote Server to Domain Controller for corp.local
Install-ADDSForest -DomainName "corp.local" -InstallDns

# 5. Force Group Policy Refresh
gpupdate /force
```
