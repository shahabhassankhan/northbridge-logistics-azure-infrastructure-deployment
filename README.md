# NorthBridge Logistics — Azure Infrastructure Deployment

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

Screenshots are stored in `/screenshots` and referenced below. Replace the filenames with your actual uploaded images.

| # | Screenshot | Description |
|---|---|---|
| 01 | `01-resource-groups.png` | Resource group list |
| 02 | `02-network-rg.png` | Networking resource group |
| 03 | `03-vnet.png` | Virtual network |
| 04 | `04-subnets.png` | Subnets |
| 05 | `05-web-nsg.png` | Web NSG |
| 06 | `06-nsg-list.png` | NSG list |
| 07 | `07-subnet-associations.png` | Subnet associations |
| 08 | `08-http-rule.png` | HTTP inbound rule |
| 09 | `09-inbound-rules.png` | Inbound rules |
| 10 | `10-vm-overview.png` | VM overview |
| 11 | `11-vm-config.png` | VM configuration |
| 12 | `12-vm-running.png` | VM running state |
| 13 | `13-vm-networking.png` | VM networking |
| 14 | `14-resource-visualizer.png` | Resource visualizer |
| 15 | `15-rdp-config.png` | RDP configuration |
| 16 | `16-website-live.png` | Website reachable externally |
| 17 | `17-storage-account.png` | Storage account |
| 18 | `18-blob-container.png` | Blob container |
| 19 | `19-uploaded-blob.png` | Uploaded blob |
| 20 | `20-recovery-vault.png` | Recovery Services Vault |
| 21 | `21-cost-management.png` | Cost analysis |
| 22 | `22-rbac.png` | RBAC role assignment |

Additional device photos taken during setup (IIS install, server desktop, live site) can go in the same folder, numbered 23-26.

## Future Improvements

- Add Log Analytics and a proper Azure Monitor workbook
- Move to an Application Gateway or Load Balancer in front of the VM
- Add a second VM and test failover
- Automate the deployment with an ARM template or Bicep instead of manual portal steps

---

Built and deployed by **Shahab Hassan Khan** as a hands-on Azure administration project.
