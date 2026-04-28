[🏠 Home](../README.md) · [Azure](README.md)

# ☁️ AZ-104 Azure Administrator — Quick Revision Cheatsheet

> **Audience:** AZ-104 candidates and Azure admins  
> **Focus:** Last-minute recall, exam traps, and decision tables  
> **Tip:** Read this after the handbook. It is meant for fast refresh, not first-time learning.

## Table of Contents

1. [Exam Weighting Snapshot](#1-exam-weighting-snapshot)
2. [Identity and Governance Cheats](#2-identity-and-governance-cheats)
3. [Storage Cheats](#3-storage-cheats)
4. [Compute Cheats](#4-compute-cheats)
5. [Networking Cheats](#5-networking-cheats)
6. [Monitoring Cheats](#6-monitoring-cheats)
7. [Common Exam Traps](#7-common-exam-traps)
8. [Quick Revision Card](#8-quick-revision-card)

## 1. Exam Weighting Snapshot

| Domain | Weight | Memory cue |
|--------|--------|------------|
| Identity and governance | 20–25% | Who and where |
| Storage | 15–20% | Data access and durability |
| Compute | 20–25% | VM, VMSS, App Service, containers |
| Networking | 15–20% | Traffic, DNS, routing, NSG |
| Monitoring | 10–15% | Metrics, logs, alerts |

## 2. Identity and Governance Cheats

| Need | Pick this |
|------|-----------|
| Human login | User |
| Group access | Group |
| External collaborator | Guest |
| App automation | Service principal or managed identity |
| Grant access at a scope | RBAC |
| Stop noncompliant deployment | Azure Policy |
| Prevent delete | Resource lock |

### Role and Scope Cheats

| Role | Use case |
|------|----------|
| Reader | View only |
| Contributor | Manage resources, not access |
| Owner | Full control and access delegation |
| User Access Administrator | Assign roles |

| Scope | Example |
|-------|---------|
| Management group | Company-wide policy |
| Subscription | Team boundary |
| Resource group | App environment |
| Resource | Single server or database |

### Governance Recall

| Control | What it is |
|---------|------------|
| Tag | Metadata for cost/ownership |
| Policy | Prevent or audit noncompliance |
| Lock | Prevent delete or modify |
| Budget | Cost guardrail |

## 3. Storage Cheats

| Topic | Quick recall |
|------|--------------|
| LRS | One datacenter, low cost |
| ZRS | Multiple zones |
| GRS | Paired region copy |
| RA-GRS | Readable secondary |

| Access method | Best for |
|--------------|-----------|
| Access key | Full account access |
| SAS | Time-bound delegated access |
| Entra auth | Identity-based access |

| Feature | Use it for |
|---------|------------|
| Soft delete | Recover deleted data |
| Snapshot | Point-in-time recovery |
| Versioning | Historical versions |
| Lifecycle rule | Tiering and expiry |

| Network access | Meaning |
|---------------|---------|
| Service endpoint | Trusted VNet path to public endpoint |
| Private endpoint | Private IP in VNet |

## 4. Compute Cheats

| Service | Best fit |
|---------|----------|
| VM | Full OS control |
| VMSS | Identical scale-out VMs |
| App Service | Managed web app hosting |
| Container Instances | Fast single container run |
| Container Apps | Managed container platform |

### VM Memory Aids

| Item | Meaning |
|------|---------|
| OS disk | Boot and OS |
| Data disk | App and data storage |
| Snapshot | Point-in-time recovery |
| Managed disk | Default Azure disk type |

| Availability model | Failure domain |
|-------------------|----------------|
| Availability set | Host/rack failure |
| Availability zone | Datacenter failure |

### App Service Aids

| Feature | Why you care |
|---------|--------------|
| Deployment slot | Safer release |
| Backup | Restore config/content |
| Custom domain + TLS | Public web app |
| VNet integration | Private connectivity |

## 5. Networking Cheats

| Component | Recall |
|-----------|--------|
| VNet | Private address space |
| Subnet | Segmented VNet slice |
| Peering | VNet-to-VNet private link |
| UDR | Custom route table |
| Public IP | Internet endpoint |
| NSG | Traffic allow/deny filter |
| ASG | Group NICs for NSG rules |
| Bastion | RDP/SSH without public IP |

### NSG and DNS Cheats

| Item | Recall |
|------|--------|
| Lower NSG priority number | Wins |
| Effective rules | Actual applied rules |
| Private endpoint DNS | Usually required |
| Azure DNS | Public zone hosting |
| Private DNS zone | Internal name resolution |

### Load Balancing Cheats

| Service | Use case |
|---------|----------|
| Load Balancer | Layer 4 distribution |
| Health probe | Backend health signal |

## 6. Monitoring Cheats

| Telemetry | Use it for |
|----------|------------|
| Metrics | Trend and threshold alerts |
| Logs | Detailed troubleshooting |
| Activity log | Control-plane audit |

| Component | Purpose |
|----------|---------|
| Diagnostic settings | Send telemetry out |
| Log Analytics | Query logs with KQL |
| Action group | Notify the right people |
| Alert processing rule | Suppress or route alerts |

## 7. Common Exam Traps

| Trap | Better answer |
|------|---------------|
| Policy grants access | No, RBAC grants access |
| Lock grants access | No, lock only protects from change/delete |
| SAS equals access key | No, SAS is scoped and temporary |
| Stopped VM equals deallocated VM | No, deallocated is the important billing/state clue |
| Private endpoint works without DNS | Not usually |
| NSG order is arbitrary | Lower number = higher priority |
| Metrics and logs are the same | They answer different questions |
| Availability set = zone | Different failure domains |
| RBAC scope does not matter | Scope is critical |

## 8. Quick Revision Card

```
  ┌──────────────────────────────────────────────────────────┐
  │  AZ-104 ONE-LINER MEMORY CARD                            │
  │                                                          │
  │  Entra ID   = who you are                                │
  │  RBAC       = what you can do and where                  │
  │  Policy     = what is allowed                            │
  │  Lock       = what must not change or delete             │
  │  Tags       = how you classify and report                │
  │  SAS        = temporary delegated storage access         │
  │  Private EP  = private IP + DNS for PaaS                 │
  │  ASG        = group NICs for NSG rules                   │
  │  Bastion    = admin access without public IP             │
  │  Set        = host/rack failure protection               │
  │  Zone       = datacenter failure protection              │
  │  VMSS       = repeatable scale-out compute               │
  │  App Service= managed web hosting                        │
  │  Metrics    = fast numeric signal                        │
  │  Logs       = deep investigation                         │
  │  Activity   = control-plane audit trail                  │
  └──────────────────────────────────────────────────────────┘
```
