# NorthBridge Logistics — Azure Infrastructure Deployment

A production-style Azure environment built for a fictional logistics company, covering networking, compute, storage, backup, and access control. The company doesn't exist. The infrastructure does.

**Author:** Shahab Hassan Khan# NorthBridge Logistics — Azure Infrastructure Deployment

A production-style Azure environment built for a fictional logistics company, covering networking, compute, storage, backup, and access control. The company doesn't exist. The infrastructure does.

**Author:** Shahab Hassan Khan

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Azure Services Used](#azure-services-used)
- [Resource Groups](#resource-groups)
- [Networking](#networking)
- [Virtual Machine](#virtual-machine)
- [Web Hosting (IIS)](#web-hosting-iis)
- [Storage](#storage)
- [Backup](#backup)
- [Access Control (RBAC)](#access-control-rbac)
- [Cost Management](#cost-management)
- [Troubleshooting & Lessons Learned](#troubleshooting--lessons-learned)
- [Skills Demonstrated](#skills-demonstrated)
- [Screenshots](#screenshots)
- [Future Improvements](#future-improvements)

---

## Overview

NorthBridge Logistics is a made-up company created as a reason to build a real Azure deployment. The goal was to design and manage infrastructure the way an Azure administrator would, not to build a working web app.

The project covers:

- Deploying and securing a Windows Server virtual machine
- Hosting a website on IIS
- Structuring resources across separate resource groups
- Setting up storage, backup, and access control
- Diagnosing and fixing a real connectivity issue during deployment

## Architecture

```
Internet
   │
   ▼
Azure Public IP Address
   │
   ▼
Network Security Group (Web NSG)
   • Allow HTTP (80)
   • Allow RDP (3389)
   │
   ▼
┌─────────────────────────────────────────┐
│        Azure Virtual Network             │
│      nbl-vnet-core (10.20.0.0/16)        │
│                                           │
│  ┌──────────────────────────────────┐   │
│  │ Web Subnet                        │   │
│  │ nbl-web-snet (10.20.1.0/24)       │   │
│  │                                    │   │
│  │ Windows Server 2016 VM            │   │
│  │ nbl-web-vm01                      │   │
│  │ IIS Web Server                    │   │
│  │ NorthBridge Logistics Website     │   │
│  └──────────────────────────────────┘   │
│                                           │
│  ┌──────────────────────────────────┐   │
│  │ App Subnet                        │   │
│  │ nbl-app-snet — reserved           │   │
│  └──────────────────────────────────┘   │
│                                           │
│  ┌──────────────────────────────────┐   │
│  │ Database Subnet                   │   │
│  │ nbl-db-snet — reserved            │   │
│  └──────────────────────────────────┘   │
│                                           │
│  AzureBastionSubnet — reserved           │
└─────────────────────────────────────────┘
   │
   ├──────────────────────┬──────────────────────┐
   ▼                                              ▼
Azure Storage Account                    Recovery Services Vault
nblstorage01                             nbl-backup-rsv01
   │                                              │
   ▼                                              ▼
Blob Container (website-assets)          Scheduled VM Backup
   │
   ▼
Images / Documents / Assets
```

**Management & Governance Layer**

- Resource Groups — `nbl-network-rg`, `nbl-compute-rg`, `nbl-storage-rg`, `nbl-operations-rg`
- RBAC — Owner, Contributor, Reader
- Azure Monitor — Metrics, Activity Logs, Alerts (concept)
- Azure Cost Management

## Azure Services Used

- Resource Groups
- Virtual Network & Subnets
- Network Security Groups
- Windows Server Virtual Machine
- Public IP Address
- Azure Storage Account & Blob Container
- Recovery Services Vault
- Role Based Access Control (RBAC)
- Azure Cost Management
- Azure Monitor (metrics, activity log, alerts)

## Resource Groups

| Resource Group | Purpose | Contains |
|---|---|---|
| `nbl-network-rg` | Networking | VNet, subnets, NSGs, public IP |
| `nbl-compute-rg` | Compute | Windows Server VM |
| `nbl-storage-rg` | Storage | Storage account, blob container |
| `nbl-operations-rg` | Operations | Recovery Services Vault |

Splitting resources this way keeps networking, compute, storage, and operations independently manageable, which is closer to how a real environment gets organized than dropping everything into one group.

## Networking

**Virtual Network:** `nbl-vnet-core` — East US 2 — `10.20.0.0/16`

| Subnet | Address Space | Purpose |
|---|---|---|
| `nbl-web-snet` | 10.20.1.0/24 | Web tier |
| `nbl-app-snet` | 10.20.2.0/24 | Application tier |
| `nbl-db-snet` | 10.20.3.0/24 | Database tier |
| `AzureBastionSubnet` | 10.20.4.0/26 | Bastion |

**Network Security Groups:** `nbl-web-nsg`, `nbl-app-nsg`, `nbl-db-nsg`

The web NSG allows RDP (port 3389) and HTTP (port 80) inbound, both scoped rules rather than a broad allow-all.

## Virtual Machine

| Property | Value |
|---|---|
| Name | `nbl-web-vm01` |
| OS | Windows Server 2016 Datacenter |
| Generation | Gen2 |
| Size | Standard_D2s_v3 |
| Region | East US 2 |
| Private IP | 10.20.1.4 |
| Public IP | 172.177.80.73 |

## Web Hosting (IIS)

IIS was installed through Server Manager as the Web Server role. The default site was verified locally at `localhost` before testing externally, and a custom HTML page for NorthBridge Logistics replaced the default IIS page.

## Storage

A general-purpose v2 storage account was created in `nbl-storage-rg` (East US 2, Standard performance tier), with a blob container named `website-assets` holding a sample logo image. Azure automatically created a logs container alongside it.

## Backup

A Recovery Services Vault was set up in `nbl-operations-rg`, with Azure Backup enabled for `nbl-web-vm01`.

## Access Control (RBAC)

Reviewed the built-in Owner, Contributor, and Reader roles and assigned Reader to demonstrate least-privilege access rather than defaulting everyone to Contributor or Owner.

## Cost Management

Reviewed spending through Azure Cost Analysis to understand where the free credit was going during the build.

## Troubleshooting & Lessons Learned

The website was unreachable from outside the VM despite everything checking out on paper: NSG rules correct, Windows Firewall allowing port 80, IIS running, and `Test-NetConnection` succeeding both locally and against the private IP.

Steps taken to isolate the problem:

- Confirmed the VM was running and reachable over RDP
- Verified the NSG rules at both subnet and NIC level
- Checked the Windows Firewall inbound rule and its network profile
- Confirmed IIS bindings were set to all unassigned addresses
- Ran `Test-NetConnection` and `netstat` to confirm the port was listening
- Tested from outside the VM's own network to rule out a local-only issue

The actual cause turned out to be simpler than any of that: the site was being tested over `https://` instead of `http://`, since 443 wasn't open. Switching to `http://172.177.80.73` resolved it immediately.

**Takeaway:** methodical, layer-by-layer troubleshooting (network, firewall, service, binding) is what actually finds the fault, even when the fault turns out to be small. Assuming the protocol before checking it costs more time than checking it first.

## Skills Demonstrated

Azure administration, Azure networking, Windows Server administration, VM deployment, network security groups, storage administration, blob storage, backup configuration, RBAC, IIS administration, cloud troubleshooting, and resource organization.

## Screenshots

All screenshots are in `/screenshots`.

| # | Screenshot | Description |
|---|---|---|
| 01 | `01_nbl_network_rg.jpg` | Networking resource group |
| 02 | `02_resources_group_list.jpg` | Full resource group list |
| 03 | `03_nbl_vnet_core.jpg` | Virtual network |
| 04 | `04_nbl_vnet_core_subnets.jpg` | Subnets |
| 05 | `05_nbl_web_nsg.jpg` | Web NSG |
| 06 | `06_network_security_group_list.jpg` | NSG list |
| 07 | `07_association_subnets_list.jpg` | Subnet associations |
| 08 | `08_allowing_http_to_web_subnet.jpg` | HTTP inbound rule |
| 09 | `09_inbound_security_rules.jpg` | Inbound security rules |
| 10 | `10_nbl_compute_vm01.jpg` | Compute resource group / VM |
| 11 | `11_nbl_web_vm01.jpg` | VM overview |
| 12 | `12_nbl_web_vm01.jpg` | VM configuration |
| 13 | `13_nbl_web_vm01.jpg` | VM running state |
| 14 | `14_nbl_web_vm01_resource_visualizer.jpg` | Resource visualizer |
| 15 | `15_nbl_web_vm01_adding_rdp_rule.jpg` | Adding RDP rule |
| 16 | `16_nbl_windows_server_main_page.jpeg` | Windows Server desktop / main page |
| 17 | `17_nbl_windows_server_adding_roles_features.jpeg` | Adding IIS role and features, step 1 |
| 18 | `18_nbl_windows_server_adding_roles_features.jpg` | Adding IIS role and features, step 2 |
| 19 | `19_nbl_windows_server_localhost.jpeg` | IIS default page confirmed on localhost |
| 20 | `20_nbl_windows_server_testing_outside_vm.jpg` | Website tested and confirmed reachable from outside the VM |
| 21 | `21_nbl_storage.jpg` | Storage account |
| 22 | `22_nbl_storage_container.jpg` | Blob container |
| 23 | `23_nbl_storage_logo_uploaded.jpg` | Uploaded blob (logo) |
| 24 | `24_nbl_backup_vm01.jpg` | Recovery Services Vault / VM backup |
| 25 | `25_nbl_cost_management.jpg` | Cost analysis |
| 26 | `26_nbl_rbac.jpg` | RBAC role assignment |

## Future Improvements

- Add Log Analytics and a proper Azure Monitor workbook
- Move to an Application Gateway or Load Balancer in front of the VM
- Add a second VM and test failover
- Automate the deployment with an ARM template or Bicep instead of manual portal steps

---

Built and deployed by **Shahab Hassan Khan** as a hands-on Azure administration project.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Azure Services Used](#azure-services-used)
- [Resource Groups](#resource-groups)
- [Networking](#networking)
- [Virtual Machine](#virtual-machine)
- [Web Hosting (IIS)](#web-hosting-iis)
- [Storage](#storage)
- [Backup](#backup)
- [Access Control (RBAC)](#access-control-rbac)
- [Cost Management](#cost-management)
- [Troubleshooting & Lessons Learned](#troubleshooting--lessons-learned)
- [Skills Demonstrated](#skills-demonstrated)
- [Screenshots](#screenshots)
- [Future Improvements](#future-improvements)

---

## Overview

NorthBridge Logistics is a made-up company created as a reason to build a real Azure deployment. The goal was to design and manage infrastructure the way an Azure administrator would, not to build a working web app.

The project covers:

- Deploying and securing a Windows Server virtual machine
- Hosting a website on IIS
- Structuring resources across separate resource groups
- Setting up storage, backup, and access control
- Diagnosing and fixing a real connectivity issue during deployment

## Architecture

```
Internet
   │
Public IP
   │
Network Security Group
   │
Virtual Machine (Windows Server 2016)
   │
IIS
   │
NorthBridge Logistics Website
   │
Azure Storage Account → Blob Container

Management layer:
Recovery Services Vault · RBAC · Azure Monitor · Cost Management
```

## Azure Services Used

- Resource Groups
- Virtual Network & Subnets
- Network Security Groups
- Windows Server Virtual Machine
- Public IP Address
- Azure Storage Account & Blob Container
- Recovery Services Vault
- Role Based Access Control (RBAC)
- Azure Cost Management
- Azure Monitor (metrics, activity log, alerts)

## Resource Groups

| Resource Group | Purpose | Contains |
|---|---|---|
| `nbl-network-rg` | Networking | VNet, subnets, NSGs, public IP |
| `nbl-compute-rg` | Compute | Windows Server VM |
| `nbl-storage-rg` | Storage | Storage account, blob container |
| `nbl-operations-rg` | Operations | Recovery Services Vault |

Splitting resources this way keeps networking, compute, storage, and operations independently manageable, which is closer to how a real environment gets organized than dropping everything into one group.

## Networking

**Virtual Network:** `nbl-vnet-core` — East US 2 — `10.20.0.0/16`

| Subnet | Address Space | Purpose |
|---|---|---|
| `nbl-web-snet` | 10.20.1.0/24 | Web tier |
| `nbl-app-snet` | 10.20.2.0/24 | Application tier |
| `nbl-db-snet` | 10.20.3.0/24 | Database tier |
| `AzureBastionSubnet` | 10.20.4.0/26 | Bastion |

**Network Security Groups:** `nbl-web-nsg`, `nbl-app-nsg`, `nbl-db-nsg`

The web NSG allows RDP (port 3389) and HTTP (port 80) inbound, both scoped rules rather than a broad allow-all.

## Virtual Machine

| Property | Value |
|---|---|
| Name | `nbl-web-vm01` |
| OS | Windows Server 2016 Datacenter |
| Generation | Gen2 |
| Size | Standard_D2s_v3 |
| Region | East US 2 |
| Private IP | 10.20.1.4 |
| Public IP | 172.177.80.73 |

## Web Hosting (IIS)

IIS was installed through Server Manager as the Web Server role. The default site was verified locally at `localhost` before testing externally, and a custom HTML page for NorthBridge Logistics replaced the default IIS page.

## Storage

A general-purpose v2 storage account was created in `nbl-storage-rg` (East US 2, Standard performance tier), with a blob container named `website-assets` holding a sample logo image. Azure automatically created a logs container alongside it.

## Backup

A Recovery Services Vault was set up in `nbl-operations-rg`, with Azure Backup enabled for `nbl-web-vm01`.

## Access Control (RBAC)

Reviewed the built-in Owner, Contributor, and Reader roles and assigned Reader to demonstrate least-privilege access rather than defaulting everyone to Contributor or Owner.

## Cost Management

Reviewed spending through Azure Cost Analysis to understand where the free credit was going during the build.

## Troubleshooting & Lessons Learned

The website was unreachable from outside the VM despite everything checking out on paper: NSG rules correct, Windows Firewall allowing port 80, IIS running, and `Test-NetConnection` succeeding both locally and against the private IP.

Steps taken to isolate the problem:

- Confirmed the VM was running and reachable over RDP
- Verified the NSG rules at both subnet and NIC level
- Checked the Windows Firewall inbound rule and its network profile
- Confirmed IIS bindings were set to all unassigned addresses
- Ran `Test-NetConnection` and `netstat` to confirm the port was listening
- Tested from outside the VM's own network to rule out a local-only issue

The actual cause turned out to be simpler than any of that: the site was being tested over `https://` instead of `http://`, since 443 wasn't open. Switching to `http://172.177.80.73` resolved it immediately.

**Takeaway:** methodical, layer-by-layer troubleshooting (network, firewall, service, binding) is what actually finds the fault, even when the fault turns out to be small. Assuming the protocol before checking it costs more time than checking it first.

## Skills Demonstrated

Azure administration, Azure networking, Windows Server administration, VM deployment, network security groups, storage administration, blob storage, backup configuration, RBAC, IIS administration, cloud troubleshooting, and resource organization.

## Screenshots

All screenshots are in `/screenshots`.

| # | Screenshot | Description |
|---|---|---|
| 01 | `01_nbl_network_rg.jpg` | Networking resource group |
| 02 | `02_resources_group_list.jpg` | Full resource group list |
| 03 | `03_nbl_vnet_core.jpg` | Virtual network |
| 04 | `04_nbl_vnet_core_subnets.jpg` | Subnets |
| 05 | `05_nbl_web_nsg.jpg` | Web NSG |
| 06 | `06_network_security_group_list.jpg` | NSG list |
| 07 | `07_association_subnets_list.jpg` | Subnet associations |
| 08 | `08_allowing_http_to_web_subnet.jpg` | HTTP inbound rule |
| 09 | `09_inbound_security_rules.jpg` | Inbound security rules |
| 10 | `10_nbl_compute_vm01.jpg` | Compute resource group / VM |
| 11 | `11_nbl_web_vm01.jpg` | VM overview |
| 12 | `12_nbl_web_vm01.jpg` | VM configuration |
| 13 | `13_nbl_web_vm01.jpg` | VM running state |
| 14 | `14_nbl_web_vm01_resource_visualizer.jpg` | Resource visualizer |
| 15 | `15_nbl_web_vm01_adding_rdp_rule.jpg` | Adding RDP rule |
| 16 | `16_nbl_windows_server_main_page.jpeg` | Windows Server desktop / main page |
| 17 | `17_nbl_windows_server_adding_roles_features.jpeg` | Adding IIS role and features, step 1 |
| 18 | `18_nbl_windows_server_adding_roles_features.jpg` | Adding IIS role and features, step 2 |
| 19 | `19_nbl_windows_server_localhost.jpeg` | IIS default page confirmed on localhost |
| 20 | `20_nbl_windows_server_testing_outside_vm.jpg` | Website tested and confirmed reachable from outside the VM |
| 21 | `21_nbl_storage.jpg` | Storage account |
| 22 | `22_nbl_storage_container.jpg` | Blob container |
| 23 | `23_nbl_storage_logo_uploaded.jpg` | Uploaded blob (logo) |
| 24 | `24_nbl_backup_vm01.jpg` | Recovery Services Vault / VM backup |
| 25 | `25_nbl_cost_management.jpg` | Cost analysis |
| 26 | `26_nbl_rbac.jpg` | RBAC role assignment |

## Future Improvements

- Add Log Analytics and a proper Azure Monitor workbook
- Move to an Application Gateway or Load Balancer in front of the VM
- Add a second VM and test failover
- Automate the deployment with an ARM template or Bicep instead of manual portal steps

---

Built and deployed by **Shahab Hassan Khan** as a hands-on Azure administration project.
