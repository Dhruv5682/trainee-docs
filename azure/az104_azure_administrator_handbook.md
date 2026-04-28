---
Version: 2.0
Last Updated: 2026-04-28
Status: Published
---

[🏠 Home](../README.md) · [Azure](README.md)

# ☁️ AZ-104 Azure Administrator — Comprehensive Study Handbook

> **Audience:** Cloud / DevOps Engineers & AZ-104 candidates
> **Focus:** Mental models, decision making, exam readiness, and production relevance
> **Source:** Microsoft Learn AZ-104 learning paths, Microsoft official exam skills outline (2026)
> **Tip:** Read section 1 first. It explains the control-plane model that everything else depends on.

---

## Table of Contents

1. [AZ-104 Exam Map and Azure Mental Model](#1-az-104-exam-map-and-azure-mental-model)
2. [Identity and Governance](#2-identity-and-governance)
3. [Storage](#3-storage)
4. [Compute](#4-compute)
5. [Virtual Networking](#5-virtual-networking)
6. [Monitoring and Maintenance](#6-monitoring-and-maintenance)
7. [Cross-Domain Decision Tables](#7-cross-domain-decision-tables)
8. [Exam Gotchas and Common Mistakes](#8-exam-gotchas-and-common-mistakes)
9. [Final Revision Card](#9-final-revision-card)

---

## 1. AZ-104 Exam Map and Azure Mental Model

The AZ-104 exam is an administrator exam, not a deep architecture exam. The job is to manage resources safely, keep access under control, and know where to look when something breaks.

### 1.1 Why AZ-104 Matters for Your Career

As a Cloud/DevOps engineer, you **will** encounter:
- Production subscriptions with hundreds of resources and unclear ownership → you need to know how governance works
- Incidents where a VM is unreachable → you need to reason about NSGs, DNS, routing, and health probes in order
- Cost surprises from forgotten resources → you need to understand tagging, budgets, and lifecycle policies
- Access requests from developers and external partners → you need to assign the right role at the right scope without over-permissioning

AZ-104 is the certification that proves you can do all of this systematically.

### 1.2 Domain Weighting

| Domain | Weight | Memory cue |
|--------|--------|------------|
| Manage Azure identities and governance | 20–25% | Who and where |
| Implement and manage storage | 15–20% | Data access and durability |
| Deploy and manage Azure compute resources | 20–25% | VM, VMSS, App Service, containers |
| Implement and manage virtual networking | 15–20% | Traffic, DNS, routing, NSG |
| Monitor and maintain Azure resources | 10–15% | Metrics, logs, alerts |

### 1.3 Control-Plane Mental Model

```
  ┌───────────────────────────────────────────────────────────────────────┐
  │                         AZURE CONTROL PLANE                          │
  │                                                                      │
  │  Microsoft Entra ID (identity layer)                                 │
  │  ┌────────────────┐   ┌────────────────────┐   ┌──────────────────┐  │
  │  │ Users/Groups   │   │  RBAC Assignments  │   │  App Identities  │  │
  │  │ Guests/Licenses│   │  at Scope          │   │  SPNs/ManagedID  │  │
  │  └───────┬────────┘   └─────────┬──────────┘   └────────┬─────────┘  │
  │          │                      │                        │           │
  │          ▼                      ▼                        ▼           │
  │  ┌───────────────────────────────────────────────────────────────┐   │
  │  │  Management Group → Subscription → Resource Group → Resource │   │
  │  └───────────────────────────────────────────────────────────────┘   │
  │          │                      │                        │           │
  │          ▼                      ▼                        ▼           │
  │  ┌────────────┐  ┌──────────────────┐  ┌──────────────────────────┐  │
  │  │   Azure    │  │   Locks (prevent │  │  Tags (classify),       │  │
  │  │   Policy   │  │   change/delete) │  │  Budgets (cost control) │  │
  │  └────────────┘  └──────────────────┘  └──────────────────────────┘  │
  └───────────────────────────────────────────────────────────────────────┘
```

The key exam idea is simple: **identity decides who can act, scope decides where they can act, policy decides what is allowed, and monitoring tells you what happened**.

### 1.4 The Four Questions to Ask

Whenever you see an AZ-104 scenario, answer these questions in order:

| Question | What you are deciding |
|----------|------------------------|
| Who? | User, group, guest, service principal, managed identity |
| Where? | Resource, resource group, subscription, management group |
| What? | Read, write, delete, assign, deploy, connect |
| Guardrails? | Policy, lock, tagging, budget, alerting |

### 1.5 Azure Resource Manager — The Single API

Every operation you perform — whether through the portal, CLI, PowerShell, REST API, or an ARM/Bicep template — goes through **Azure Resource Manager (ARM)**. This is the single control plane.

```
  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐
  │  Portal  │  │  CLI     │  │  PS      │  │  REST API  │
  └────┬─────┘  └────┬─────┘  └────┬─────┘  └─────┬──────┘
       │             │             │               │
       └─────────────┴──────┬──────┴───────────────┘
                            ▼
                ┌───────────────────────┐
                │  Azure Resource       │
                │  Manager (ARM)        │
                │  • Authentication     │
                │  • Authorization      │
                │  • Request routing    │
                └───────────┬───────────┘
                            │
          ┌─────────────────┼────────────────────┐
          ▼                 ▼                    ▼
  ┌──────────────┐  ┌──────────────┐   ┌──────────────┐
  │  Compute RP  │  │  Network RP  │   │  Storage RP  │
  │  (VMs, etc.) │  │  (VNets,etc.)│   │  (Accounts)  │
  └──────────────┘  └──────────────┘   └──────────────┘
```

**Why this matters:** When you assign RBAC, you are telling ARM "this principal is allowed to call these operations." When a policy evaluates, ARM checks it before forwarding the request to the resource provider. Everything flows through ARM.

---

## 2. Identity and Governance

### 2.1 Why Identity and Governance Matter

As an Azure administrator, you **will** encounter:
- Onboarding new team members who need the right access without seeing production secrets
- External vendors who need temporary, scoped access to specific resource groups
- Compliance audits asking "who has Owner access to production subscriptions?"
- Developers deploying resources in wrong regions or using disallowed SKUs
- Cost overruns from untagged, orphaned resources that nobody claims ownership of

Identity and governance is not bureaucracy — it is the foundation that prevents every other domain from becoming a security or cost incident.

### 2.2 Microsoft Entra ID in Plain Language

Microsoft Entra ID (formerly Azure Active Directory) is the identity directory for the tenant. It stores users, groups, service principals, and guest accounts, then uses those objects to drive authentication and authorization.

Think of it as the directory that answers: **who are you, and what are you allowed to do?**

```
  ┌───────────────────────── Microsoft Entra ID ─────────────────────────┐
  │                                                                      │
  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────────────────┐  │
  │  │  Users   │  │  Groups  │  │  Guests  │  │  App Registrations  │  │
  │  │          │  │          │  │ (B2B)    │  │  & Service Principals│  │
  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └──────────┬──────────┘  │
  │       │             │             │                    │             │
  │       └──────┬──────┴──────┬──────┘                    │             │
  │              │             │                           │             │
  │              ▼             ▼                           ▼             │
  │     ┌────────────────────────────┐          ┌──────────────────────┐ │
  │     │   RBAC Role Assignments   │          │   Managed Identities │ │
  │     │   (at a specific scope)   │          │   (system/user       │ │
  │     │                           │          │    assigned)         │ │
  │     └────────────────────────────┘          └──────────────────────┘ │
  │                                                                      │
  │  Tenant-level features:                                              │
  │  • Self-service password reset (SSPR)                                │
  │  • Multi-factor authentication (MFA)                                 │
  │  • Conditional Access policies                                       │
  │  • Group-based licensing                                             │
  └──────────────────────────────────────────────────────────────────────┘
```

### 2.3 Identity Objects — What Each One Is and When to Use It

| Object | What it is | Best use | CLI to create/manage |
|--------|------------|----------|---------------------|
| User | Human identity in the tenant | Admins, operators, trainees | `az ad user create` |
| Group | Collection of identities | Role assignment, licensing, policy targeting | `az ad group create` |
| Guest | External user invited via B2B | Vendor access, cross-org collaboration | `az ad user create --user-type Guest` |
| Service principal | App identity in the tenant | Automation, CI/CD pipelines, apps | `az ad sp create-for-rbac` |
| Managed identity (system) | Azure-managed identity tied to a resource | VM, App Service, Function — no secrets to store | Enabled per-resource |
| Managed identity (user) | Azure-managed identity you control lifecycle of | Shared across multiple resources | `az identity create` |

#### Creating and Managing Users

```bash
# Create a new user
az ad user create \
  --display-name "Jane Doe" \
  --user-principal-name jane@contoso.onmicrosoft.com \
  --password "TempP@ss123!" \
  --force-change-password-next-sign-in true

# List all users
az ad user list --output table

# Delete a user
az ad user delete --id jane@contoso.onmicrosoft.com
```

#### Creating and Managing Groups

```bash
# Create a security group
az ad group create \
  --display-name "AZ104-Lab-Team" \
  --mail-nickname "az104team"

# Add a user to the group
az ad group member add \
  --group "AZ104-Lab-Team" \
  --member-id <user-object-id>

# List group members
az ad group member list --group "AZ104-Lab-Team" --output table
```

#### Creating a Service Principal for CI/CD

```bash
# Create an SPN with Contributor role on a resource group
az ad sp create-for-rbac \
  --name "github-actions-deploy" \
  --role Contributor \
  --scopes /subscriptions/<sub-id>/resourceGroups/production-rg

# Output includes appId, password, and tenant — store these securely
```

> **Production Gotcha: Service Principal vs Managed Identity**
>
> **The Problem:** Teams create service principals with long-lived secrets for Azure-to-Azure communication (e.g., a VM calling Key Vault). The secret expires or leaks.
>
> **Why You'll Hit This:** Service principal secrets have expiry dates. If nobody rotates them, pipelines break silently at 2 AM.
>
> **The Fix:** Use managed identities for any Azure resource talking to another Azure resource. No secrets to manage, rotate, or leak.
>
> **Prevention:** Reserve service principals for external systems (GitHub Actions, Jenkins) that cannot use managed identities. For everything inside Azure, use managed identity.

### 2.4 Licensing and Group-Based Licensing

Group-based licensing is easier to maintain than assigning licenses user by user. The exam usually wants you to prefer group assignment when the requirement is consistent access for a set of users.

| Scenario | Better choice |
|----------|---------------|
| 5 admins need the same M365 license | Group-based licensing |
| One-off temporary access | Direct user assignment |
| Multiple external collaborators | Guest accounts + scoped groups |
| New hire onboarding at scale | Dynamic group + license assignment |

### 2.5 Self-Service Password Reset (SSPR) and External Users

Self-service password reset is an identity resilience feature. For the exam, remember:
- The user must be **registered** for SSPR (security questions, phone, email)
- The tenant admin must **enable** SSPR for the relevant users/groups
- SSPR can require one or two authentication methods
- Guest users **cannot** use SSPR in your tenant — they reset passwords in their home tenant

### 2.6 Azure RBAC — The Authorization Model

Azure RBAC is authorization at scope. It answers: **who can do what, and where?**

```
  ┌───────────────────────────────────────────────────────────────────┐
  │                   RBAC ASSIGNMENT MODEL                          │
  │                                                                   │
  │   Permission = Role Definition + Scope + Security Principal       │
  │                                                                   │
  │   ┌───────────────┐       ┌──────────────┐       ┌────────────┐  │
  │   │  Principal     │──gets─►│  Role         │──at──►│  Scope     │  │
  │   │  (user/group/  │       │  (Reader,     │       │  (MG/Sub/  │  │
  │   │   SPN/MI)      │       │   Contributor,│       │   RG/Res)  │  │
  │   └───────────────┘       │   Owner, etc.) │       └────────────┘  │
  │                           └──────────────┘                        │
  │                                                                   │
  │   Inheritance flows DOWN:                                         │
  │   Management Group → Subscription → Resource Group → Resource     │
  │   (a role at subscription applies to ALL resource groups below)   │
  └───────────────────────────────────────────────────────────────────┘
```

#### Scope Hierarchy

| Scope | Typical use | Example |
|-------|-------------|---------|
| Management group | Enforce rules across multiple subscriptions | Company-wide audit policy |
| Subscription | Broad team or project-level access | Dev team gets Contributor on dev sub |
| Resource group | Common app or environment boundary | App team manages their own RG |
| Resource | Narrowest control for a single resource | DBA gets access to one SQL server |

#### Built-in Roles — The Four You Must Know

| Role | Practical meaning | Common exam trap |
|------|-------------------|------------------|
| Reader | View resources, cannot change anything | Cannot start/stop VMs |
| Contributor | Create, update, delete resources — but **cannot** assign roles | Cannot grant others access |
| Owner | Full control including access management | Use sparingly — blast radius is large |
| User Access Administrator | Can manage role assignments but not resources | Often the real answer when question says "manage access" |

> **Exam Trap:** The exam often tempts you with Contributor when the real need is **User Access Administrator** or **Owner**. If the question says "allow a user to assign roles to others," Contributor is wrong — it cannot manage RBAC.

#### Assigning Roles via CLI

```bash
# Assign Contributor to a group at resource group scope
az role assignment create \
  --assignee-object-id <group-object-id> \
  --assignee-principal-type Group \
  --role "Contributor" \
  --scope /subscriptions/<sub-id>/resourceGroups/az104-lab-rg

# List all role assignments for a resource group
az role assignment list \
  --resource-group az104-lab-rg \
  --output table

# Remove a role assignment
az role assignment delete \
  --assignee <object-id> \
  --role "Contributor" \
  --scope /subscriptions/<sub-id>/resourceGroups/az104-lab-rg
```

### 2.7 Custom RBAC Roles

When built-in roles are too broad or too narrow, create a custom role. Custom roles are defined as JSON with a list of allowed actions and a set of assignable scopes.

```bash
# List actions available for a resource provider
az provider operation list --namespace Microsoft.Storage --output table

# Create a custom role from JSON definition
az role definition create --role-definition '{
  "Name": "Storage Blob Reader Only",
  "Description": "Can read blobs but not list keys or manage accounts",
  "Actions": [
    "Microsoft.Storage/storageAccounts/blobServices/containers/read",
    "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
  ],
  "NotActions": [],
  "DataActions": [
    "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
  ],
  "AssignableScopes": [
    "/subscriptions/<sub-id>"
  ]
}'

# List custom roles
az role definition list --custom-role-only true --output table
```

**Key distinctions for the exam:**

| Concept | What it means |
|---------|---------------|
| Actions | Control-plane operations (manage resources) |
| DataActions | Data-plane operations (read/write actual data) |
| NotActions | Excluded operations from Actions |
| AssignableScopes | Where the custom role can be assigned |

### 2.8 Azure Policy — Enforcing Standards

Azure Policy evaluates resources against rules and can **deny**, **audit**, or **modify** noncompliant deployments. Policy does **not** grant access — that is RBAC's job.

```
  ┌───────────────────────────────────────────────────────────────┐
  │                    AZURE POLICY FLOW                          │
  │                                                               │
  │   Deployment request                                          │
  │        │                                                      │
  │        ▼                                                      │
  │   ┌──────────────────┐                                        │
  │   │  ARM receives    │                                        │
  │   │  the request     │                                        │
  │   └────────┬─────────┘                                        │
  │            │                                                  │
  │            ▼                                                  │
  │   ┌──────────────────┐     ┌──────────────────────────────┐   │
  │   │  RBAC check      │────►│  Does the principal have     │   │
  │   │  (authorization) │     │  permission? YES → continue  │   │
  │   └────────┬─────────┘     └──────────────────────────────┘   │
  │            │                                                  │
  │            ▼                                                  │
  │   ┌──────────────────┐     ┌──────────────────────────────┐   │
  │   │  Policy check    │────►│  Is this resource compliant? │   │
  │   │  (compliance)    │     │  YES → deploy                │   │
  │   └────────┬─────────┘     │  NO  → deny / audit / modify│   │
  │            │               └──────────────────────────────┘   │
  │            ▼                                                  │
  │   Resource is created (or denied)                             │
  └───────────────────────────────────────────────────────────────┘
```

#### Common Built-in Policies for AZ-104

| Policy | Effect | Use case |
|--------|--------|----------|
| Allowed locations | Deny | Restrict deployments to approved regions |
| Allowed resource types | Deny | Block expensive or unapproved services |
| Require a tag on resources | Deny | Enforce cost-center tagging |
| Inherit a tag from the resource group | Modify | Auto-apply environment tags |
| Allowed VM SKUs | Deny | Prevent oversized VM deployments |

#### Assigning Policy via CLI

```bash
# List built-in policy definitions
az policy definition list \
  --query "[?displayName=='Allowed locations']" \
  --output table

# Assign the "Allowed locations" policy to a resource group
az policy assignment create \
  --name "restrict-to-eastus" \
  --display-name "Only allow East US deployments" \
  --policy "/providers/Microsoft.Authorization/policyDefinitions/<definition-id>" \
  --scope /subscriptions/<sub-id>/resourceGroups/az104-lab-rg \
  --params '{"listOfAllowedLocations": {"value": ["eastus", "eastus2"]}}'

# Check compliance state
az policy state summarize \
  --resource-group az104-lab-rg
```

#### Policy Initiatives (Policy Sets)

An initiative groups multiple policies together. Instead of assigning 10 policies individually, assign one initiative.

```bash
# List built-in initiatives
az policy set-definition list --query "[?contains(displayName, 'ISO')]" --output table
```

### 2.9 Management Groups, Subscriptions, and Resource Groups

Use this mental order — each level is a scope boundary:

```
  ┌────────────────────────────────────────────────────────────────┐
  │                    AZURE SCOPE HIERARCHY                       │
  │                                                                │
  │   Tenant Root Management Group                                 │
  │   └── Management Group (e.g., "Production")                   │
  │       ├── Subscription A (billing boundary)                    │
  │       │   ├── Resource Group: web-app-rg (lifecycle container) │
  │       │   │   ├── App Service                                  │
  │       │   │   ├── SQL Database                                 │
  │       │   │   └── Storage Account                              │
  │       │   └── Resource Group: monitoring-rg                    │
  │       │       └── Log Analytics Workspace                      │
  │       └── Subscription B                                       │
  │           └── Resource Group: dev-rg                           │
  │               └── VM + VNet                                    │
  └────────────────────────────────────────────────────────────────┘
```

| Level | What it is | Key fact |
|-------|------------|----------|
| Management group | Organizes subscriptions | Policy and RBAC applied here cascade to all child subscriptions |
| Subscription | Billing and access boundary | Each subscription has its own resource limits (quotas) |
| Resource group | Lifecycle container | Deleting a resource group deletes **everything** inside it |
| Resource | The actual Azure object | Smallest unit of management |

#### Managing Resource Groups

```bash
# Create a resource group
az group create --name az104-lab-rg --location eastus

# List resource groups
az group list --output table

# Tag a resource group
az group update \
  --name az104-lab-rg \
  --set tags.environment=lab tags.owner=vishvam tags.costcenter=training

# Delete a resource group (deletes ALL resources inside)
az group delete --name az104-lab-rg --yes --no-wait
```

### 2.10 Resource Locks

Locks prevent accidental deletion or modification. They apply regardless of RBAC — even an Owner must remove the lock before deleting.

| Lock type | What it blocks |
|-----------|---------------|
| CanNotDelete | Resource can be modified but not deleted |
| ReadOnly | Resource cannot be modified or deleted |

```bash
# Add a CanNotDelete lock
az lock create \
  --name prevent-delete \
  --lock-type CanNotDelete \
  --resource-group az104-lab-rg

# Add a ReadOnly lock to a specific resource
az lock create \
  --name freeze-config \
  --lock-type ReadOnly \
  --resource-group az104-lab-rg \
  --resource-name my-storage \
  --resource-type Microsoft.Storage/storageAccounts

# List locks
az lock list --resource-group az104-lab-rg --output table

# Remove a lock (required before you can delete the resource)
az lock delete --name prevent-delete --resource-group az104-lab-rg
```

> **Production Gotcha: ReadOnly Locks Are Stricter Than You Think**
>
> **The Problem:** You add a ReadOnly lock to a storage account to prevent changes. Now nobody can upload blobs, generate SAS tokens, or rotate keys — because those are all "write" operations.
>
> **Why You'll Hit This:** ReadOnly locks block data-plane writes too, not just control-plane changes.
>
> **The Fix:** Use CanNotDelete instead of ReadOnly when you want to protect the resource's existence but still allow normal operations.

### 2.11 Tags — Classify, Report, and Automate

Tags are key-value metadata on resources. They do not enforce anything — they are for organization, cost reporting, and automation targeting.

```bash
# Tag a resource
az resource tag \
  --ids /subscriptions/<sub-id>/resourceGroups/az104-lab-rg/providers/Microsoft.Compute/virtualMachines/my-vm \
  --tags environment=production team=backend costcenter=CC-1234

# List all resources with a specific tag
az resource list --tag environment=production --output table
```

**Common tagging strategy:**

| Tag key | Purpose | Example value |
|---------|---------|---------------|
| `environment` | Lifecycle stage | `production`, `staging`, `dev` |
| `owner` | Responsible person/team | `vishvam`, `backend-team` |
| `costcenter` | Billing allocation | `CC-1234` |
| `project` | Project association | `customer-portal` |
| `auto-shutdown` | Automation trigger | `true` |

### 2.12 Cost Management and Budgets

Azure Cost Management helps you track spending and set guardrails. For the exam, understand:

```bash
# Create a budget on a resource group
az consumption budget create \
  --budget-name "lab-budget" \
  --amount 50 \
  --time-grain Monthly \
  --start-date 2026-05-01 \
  --end-date 2026-12-31 \
  --resource-group az104-lab-rg \
  --category Cost
```

| Concept | What it does |
|---------|-------------|
| Cost analysis | View spending breakdown by resource, tag, region |
| Budget | Set a spending threshold with alerts |
| Budget alert | Notify via action group when a percentage of budget is reached |
| Cost allocation | Assign shared costs to specific teams via tags |
| Advisor recommendations | Azure's built-in suggestions for right-sizing and savings |

### 2.13 Policy vs RBAC vs Locks — Decision Table

| Tool | Question answered | What it does NOT do |
|------|-------------------|---------------------|
| RBAC | Who can act? | Does not define what resource types are allowed |
| Policy | What is allowed? | Does not grant access to anyone |
| Lock | Can this be changed or deleted? | Does not grant or deny access |
| Tags | How do we classify? | Does not enforce or restrict anything |
| Budget | How much can we spend? | Does not block deployments (only alerts) |

> **Governance Gotcha: Policy Does Not Grant Access**
>
> **The Problem:** A common exam trap is assuming policy grants access. It does not. A user can have RBAC permission and still be blocked by policy, or they can have policy compliance and still lack RBAC access.
>
> **Mental Model:** RBAC is the gate (who gets in), Policy is the rulebook (what is allowed inside), Locks are the safety bolts (what cannot be removed), Tags are the labels (how we find things).

---

