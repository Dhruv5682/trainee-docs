---
Version: 2.0
Last Updated: 2026-04-28
Status: Published
---

[🏠 Home](../README.md) · [Azure](README.md)

# ☁️ AZ-104 Azure Administrator — Quick Revision Cheatsheet

> **For:** AZ-104 candidates and Azure admins
> **Purpose:** Fast lookup tables only — no explanations. Read the [handbook](az104_azure_administrator_handbook.md) first.
> **💡 Tip:** Use TOC to jump to sections. Use Ctrl+F to search for a keyword.

---

## Table of Contents

1. [Exam Weighting](#1-exam-weighting)
2. [Identity & Governance](#2-identity--governance)
3. [Storage](#3-storage)
4. [Compute](#4-compute)
5. [Networking](#5-networking)
6. [Monitoring](#6-monitoring)
7. [Key CLI Commands](#7-key-cli-commands)

---

## 1. Exam Weighting

| Domain | Weight |
|--------|--------|
| Identity and governance | 20–25% |
| Storage | 15–20% |
| Compute | 20–25% |
| Networking | 15–20% |
| Monitoring | 10–15% |

---

## 2. Identity & Governance

### Identity Objects

| Object | Use |
|--------|-----|
| User | Human identity |
| Group | Batch role/license assignment |
| Guest | External B2B collaborator |
| Service principal | CI/CD, external automation |
| Managed identity (system) | Azure resource → Azure resource (no secrets) |
| Managed identity (user) | Shared across multiple resources |

### RBAC Roles

| Role | Can manage resources | Can assign roles |
|------|---------------------|-----------------|
| Reader | View only | ✗ |
| Contributor | Yes | ✗ |
| Owner | Yes | ✅ |
| User Access Admin | ✗ | ✅ |

### RBAC Scope Hierarchy

| Level | Example |
|-------|---------|
| Management group | Company-wide policy |
| Subscription | Team/project boundary |
| Resource group | App environment |
| Resource | Single server or DB |

### Custom Roles

| Field | Purpose |
|-------|---------|
| Actions | Control-plane operations |
| DataActions | Data-plane operations |
| NotActions | Excluded operations |
| AssignableScopes | Where role can be used |

### Governance Tools

| Tool | Purpose |
|------|---------|
| Policy (deny) | Block noncompliant deployments |
| Policy (audit) | Flag without blocking |
| Initiative | Group of policies |
| Lock (CanNotDelete) | Prevent deletion |
| Lock (ReadOnly) | Prevent all changes (including data-plane) |
| Tag | Metadata for cost/ownership |
| Budget | Spending threshold + alerts |

---

## 3. Storage

### Redundancy

| Type | Copies | Protects against |
|------|--------|-----------------|
| LRS | 3 (one zone) | Drive/rack failure |
| ZRS | 3 (three zones) | Datacenter failure |
| GRS | 6 (two regions) | Regional disaster |
| RA-GRS | 6 + read secondary | Regional disaster + read access |

### Blob Tiers

| Tier | Access cost | Storage cost | Retrieval |
|------|-----------|-------------|-----------|
| Hot | Lowest | Highest | Instant |
| Cool (≥30d) | Higher | Lower | Instant |
| Cold (≥90d) | Very High | Very Low | Instant |
| Archive (≥180d) | Highest | Lowest | Hours (rehydrate) |

### Access Methods

| Method | Scope | Duration |
|--------|-------|----------|
| Account keys | Entire account | Permanent until rotated |
| SAS token | Specific resource/permissions | Time-bound |
| Entra ID + RBAC | Identity-based | Persistent |
| Stored access policy | Group SAS tokens | Revocable |

### Network Access

| Approach | DNS change needed | Data path |
|----------|------------------|-----------|
| Public (all networks) | No | Internet |
| Service endpoint | No | Azure backbone |
| Private endpoint | Yes (Private DNS zone) | Fully private |

### Data Protection

| Feature | Use case |
|---------|----------|
| Soft delete | Recover deleted blobs |
| Versioning | Historical versions |
| Point-in-time restore | Roll back container state |
| Immutability (WORM) | Legal compliance |
| Lifecycle rules | Auto-tier or expire |

### Transfer Tools

| Tool | Best for |
|------|----------|
| AzCopy | CLI bulk transfer |
| Storage Explorer | GUI management |
| Data Box | Petabyte-scale offline |
| Azure File Sync | Hybrid file caching |

---

## 4. Compute

### Service Selection

| Service | Best fit |
|---------|----------|
| VM | Full OS control |
| VMSS | Scale-out identical VMs |
| App Service | Managed web apps/APIs |
| ACI | Quick single container |
| Container Apps | Event-driven microservices |
| ACR | Private Docker registry |

### VM Disk Types

| Type | Best for |
|------|----------|
| Ultra Disk | Top-tier databases |
| Premium SSD v2 | High-perf databases |
| Premium SSD | Production workloads |
| Standard SSD | Dev/test |
| Standard HDD | Backups, archival |

### Availability

| Model | Protects against | SLA |
|-------|-----------------|-----|
| Single VM (Premium) | — | 99.9% |
| Availability Set | Rack/host failure | 99.95% |
| Availability Zone | Datacenter failure | 99.99% |

### App Service Tiers

| Tier | Key features |
|------|-------------|
| Free/Shared | Learning (no SLA) |
| Basic | Custom domains, SSL |
| Standard | Auto-scale, slots, backups |
| Premium | VNet integration, high perf |
| Isolated | Dedicated ASE |

### ARM Template Sections

| Section | Purpose |
|---------|---------|
| parameters | Deploy-time inputs |
| variables | Reusable internal values |
| resources | Azure objects to create |
| outputs | Post-deployment return values |

---

## 5. Networking

### Core Components

| Component | Purpose |
|-----------|---------|
| VNet | Private address space |
| Subnet | VNet segment |
| NSG | Traffic allow/deny filter |
| ASG | Group NICs for rules |
| Peering | VNet-to-VNet private link |
| UDR | Custom route override |
| Public IP | Internet endpoint |

### Reserved Subnet Names

| Name | For |
|------|-----|
| `AzureBastionSubnet` | Bastion host (min /27) |
| `GatewaySubnet` | VPN/ExpressRoute gateway |

### NSG Defaults (cannot delete)

| Priority | Rule | Action |
|----------|------|--------|
| 65000 | AllowVnetInBound | Allow |
| 65001 | AllowAzureLBInBound | Allow |
| 65500 | DenyAllInBound | Deny |

### Load Balancing

| Service | Layer | Key feature |
|---------|-------|-------------|
| Load Balancer | L4 (TCP/UDP) | Port-based distribution |
| Application Gateway | L7 (HTTP) | URL routing, WAF, SSL termination |

### Connectivity

| Service | Path | Best for |
|---------|------|----------|
| VNet Peering | Azure backbone | VNet-to-VNet |
| VPN Gateway | Internet (encrypted) | On-prem hybrid |
| ExpressRoute | Private provider | Enterprise hybrid |
| Bastion | Portal | Secure RDP/SSH |

### Endpoints

| Type | DNS change | Access path |
|------|-----------|-------------|
| Service endpoint | No | Azure backbone (public IP) |
| Private endpoint | Yes | Private IP in VNet |

### Network Watcher Tools

| Tool | Use |
|------|-----|
| IP Flow Verify | NSG allow/deny check |
| Next Hop | Route decision |
| Connection Troubleshoot | End-to-end test |
| NSG Flow Logs | Traffic audit |
| Packet Capture | Deep debug |

---

## 6. Monitoring

### Telemetry Types

| Type | Speed | Best for |
|------|-------|----------|
| Metrics | Real-time | Threshold alerts, dashboards |
| Logs | Query-time | Deep investigation (KQL) |
| Activity Log | Near real-time | Control-plane audit |

### Alert Components

| Component | Role |
|-----------|------|
| Alert rule | Detect condition |
| Action group | Notify (email, SMS, webhook) |
| Alert processing rule | Suppress or route |

### Alert Types

| Type | Trigger |
|------|---------|
| Metric | Numeric threshold |
| Log | KQL query match |
| Activity log | Resource operation |
| Service health | Platform issue |

### KQL Quick Reference

| Operator | Does |
|----------|------|
| `where` | Filter |
| `summarize` | Aggregate |
| `project` | Select columns |
| `order by` | Sort |
| `bin()` | Time buckets |
| `ago()` | Relative time |
| `render` | Chart |

### Azure Backup

| Component | Purpose |
|-----------|---------|
| Recovery Services Vault | Stores backups |
| Backup policy | Schedule + retention |
| Recovery point | Restorable snapshot |
| Soft delete (14d) | Protect against accidental delete |

---

## 7. Key CLI Commands

### Identity

```bash
az ad user create --display-name "..." --user-principal-name "..."
az ad group create --display-name "..." --mail-nickname "..."
az ad sp create-for-rbac --name "..." --role Contributor --scopes "..."
az role assignment create --assignee-object-id "..." --role "..." --scope "..."
az role definition create --role-definition @role.json
```

### Governance

```bash
az policy assignment create --name "..." --policy "..." --scope "..."
az lock create --name "..." --lock-type CanNotDelete --resource-group "..."
az group update --name "..." --set tags.env=prod tags.owner=team
```

### Storage

```bash
az storage account create --name "..." --sku Standard_ZRS --kind StorageV2
az storage container create --name "..." --account-name "..."
az storage blob generate-sas --account-name "..." --permissions r --expiry "..."
```

### Compute

```bash
az vm create --name "..." --image Ubuntu2204 --generate-ssh-keys
az vm deallocate --name "..."               # Stop billing
az vm disk attach --vm-name "..." --new --size-gb 128
az vmss create --name "..." --instance-count 2 --upgrade-policy-mode automatic
az webapp create --name "..." --plan "..."
az webapp deployment slot create --name "..." --slot staging
az webapp deployment slot swap --name "..." --slot staging
```

### Networking

```bash
az network vnet create --name "..." --address-prefix 10.0.0.0/16 --subnet-name "..."
az network nsg create --name "..."
az network nsg rule create --nsg-name "..." --priority 100 --access Allow --protocol Tcp
az network lb create --name "..." --sku Standard
az network dns zone create --name "..."
az network private-dns zone create --name "..."
az network watcher test-ip-flow --vm "..." --direction Inbound --protocol TCP
```

### Monitoring

```bash
az monitor log-analytics workspace create --workspace-name "..."
az monitor diagnostic-settings create --resource "..." --workspace "..."
az monitor action-group create --name "..." --action email ops ops@company.com
az monitor metrics alert create --name "..." --condition "avg Percentage CPU > 80"
az backup vault create --name "..."
az backup protection enable-for-vm --vault-name "..." --vm "..."
```

---

**Version:** 2.0 | **Last Updated:** 2026-04-28
