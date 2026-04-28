[🏠 Home](../README.md) · [Azure](README.md)

# ☁️ AZ-104 Ultra-Short Cram Sheet

> **Audience:** Exam-day revision only  
> **Focus:** What to remember when you have 5 minutes, not 5 hours  
> **Tip:** If a question feels vague, map it to who, scope, allowed, network, or telemetry.

## 1. One-Line Model

**Identity decides who, scope decides where, policy decides what is allowed, networking decides how it reaches, and monitoring tells you what happened.**

## 2. Fast Recall Table

| Topic | Memory cue |
|------|------------|
| Entra ID | Who you are |
| RBAC | What you can do and where |
| Policy | What is allowed |
| Lock | What must not change or delete |
| Tag | How you classify resources |
| SAS | Temporary delegated storage access |
| Private endpoint | Private IP + DNS for PaaS |
| Service endpoint | Trusted VNet path to public PaaS |
| VM | Full OS control |
| VMSS | Repeatable scale-out VMs |
| App Service | Managed web hosting |
| NSG | Traffic filter |
| Bastion | Admin access without public IP |
| Metrics | Fast alerts and trends |
| Logs | Deep investigation |
| Activity log | Control-plane audit trail |

## 3. Highest-Yield Comparisons

| If the question says... | Think... |
|-------------------------|----------|
| Temporary access to storage | SAS |
| Identity-based storage access | Entra auth |
| Prevent delete | Lock |
| Stop noncompliant deployment | Policy |
| Assign permissions | RBAC |
| Private access to PaaS | Private endpoint + DNS |
| Need many identical VMs | VMSS |
| Need a managed web app | App Service |
| Need secure admin access | Bastion |

## 4. Common Traps

| Trap | Correct answer |
|------|----------------|
| Policy grants access | No, RBAC grants access |
| Lock grants access | No, lock only protects |
| SAS equals access key | No, SAS is scoped and time-bound |
| Stopped VM equals deallocated VM | No, deallocated is the billing/state clue |
| Private endpoint works without DNS | Usually false |
| Metrics and logs are the same | They solve different problems |

## 5. 30-Second Card

```
Entra = who
RBAC = where + what you can do
Policy = what is allowed
Lock = what is protected
SAS = temporary storage access
Private EP = private IP + DNS
VMSS = scale-out VMs
App Service = managed web app
NSG = packet filter
Bastion = safe admin access
Metrics = trends
Logs = investigation
```
