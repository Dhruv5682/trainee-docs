---
Version: 2.0
Last Updated: 2026-04-28
Status: Published
---

[🏠 Home](../README.md) · [Azure](README.md)

# ☁️ AZ-104 Ultra-Short Cram Sheet

> **When:** Exam-day, 5 minutes before you start
> **How:** Read the one-liner model, scan the "if you see X → think Y" table, glance at traps. Go.

---

## The Model

**Identity decides who → scope decides where → policy decides what is allowed → networking decides how it reaches → monitoring tells you what happened.**

## If You See This → Think This

| Question pattern | Answer direction |
|-----------------|------------------|
| "Temporary access to storage" | SAS token |
| "Identity-based storage access" | Entra ID + data RBAC role |
| "Prevent accidental delete" | Lock (CanNotDelete) |
| "Stop noncompliant deployment" | Azure Policy (deny) |
| "Assign permissions to others" | User Access Admin or Owner (NOT Contributor) |
| "Private access to PaaS" | Private endpoint + Private DNS zone |
| "Many identical VMs, auto-scale" | VMSS |
| "Managed web app with zero-downtime deploy" | App Service + deployment slot swap |
| "Secure RDP/SSH without public IP" | Azure Bastion |
| "Route /api to backend A, /images to backend B" | Application Gateway (L7) |
| "Distribute TCP traffic across VMs" | Load Balancer (L4) |
| "Connect on-prem to Azure" | VPN Gateway (internet) or ExpressRoute (private) |
| "VM not reachable after NSG change" | Network Watcher → IP Flow Verify |
| "Who deleted resource X?" | Activity Log |
| "CPU trending high" | Metrics alert |
| "Deep investigation of app errors" | Logs + KQL in Log Analytics |
| "No data in Log Analytics" | Check diagnostic settings on the source resource |
| "Back up a VM" | Recovery Services Vault + backup policy |
| "Data must not be deleted for 7 years" | Immutability policy (WORM) |
| "Move old data to cheaper storage" | Lifecycle management rule |

## Traps — Read Once, Remember

| Wrong assumption | Reality |
|-----------------|---------|
| Policy = access | Policy = compliance. RBAC = access. |
| Contributor can assign roles | It cannot. |
| ReadOnly lock = safe to use everywhere | It blocks data-plane writes (blob uploads, key rotation). |
| Stopped VM = no billing | Only **deallocated** stops compute billing. |
| SAS = access key | SAS is scoped + time-bound. Keys are root access. |
| Archive = instant | Rehydration = up to 15 hours. |
| Private endpoint just works | Needs Private DNS zone or it resolves to public IP. |
| NSG rules apply randomly | Lower priority number wins. |
| Health probe pass = app healthy | Probe checks what you configure, not app logic. |
| VNet peering is transitive | A↔B + B↔C ≠ A↔C |

## 30-Second Card

```
Entra   = who           RBAC    = what + where
Policy  = what allowed  Lock    = what protected
SAS     = temp access   Private EP = private IP + DNS
VMSS    = scale VMs     App Svc = managed web
NSG     = packet filter Bastion = safe admin
LB      = L4 traffic    App GW  = L7 + WAF
VPN GW  = hybrid tunnel ExpressRoute = private link
Metrics = fast trends   Logs    = deep query (KQL)
Activity= audit trail   Backup  = vault + policy
```

---

**Version:** 2.0 | **Last Updated:** 2026-04-28
