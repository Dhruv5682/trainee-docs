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

## 3. Storage

### 3.1 Why Storage Matters

As an Azure administrator, you **will** encounter:
- Application teams needing to store terabytes of unstructured data (logs, images, backups) cheaply.
- Security teams demanding that data at rest is encrypted and not exposed to the public internet.
- Legal or compliance requirements dictating that data is retained for 7 years and cannot be deleted (immutability).
- Cost audits revealing that hot storage is being used for data that hasn't been accessed in years.

Storage in Azure is about balancing **durability (redundancy)**, **performance (tiers)**, **access (security)**, and **cost**.

### 3.2 Storage Account Mental Model

A storage account is a top-level container for your data.

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                        Storage Account                           │
  │  (defines region, redundancy, performance tier, network rules)   │
  │                                                                  │
  │  ┌───────────────┐ ┌───────────────┐ ┌───────────┐ ┌───────────┐ │
  │  │ Blob Service  │ │ File Service  │ │ Queue     │ │ Table     │ │
  │  │ (Unstructured)│ │ (SMB Shares)  │ │ Service   │ │ Service   │ │
  │  └──────┬────────┘ └──────┬────────┘ └───────────┘ └───────────┘ │
  │         │                 │                                      │
  │  ┌──────▼───────┐  ┌──────▼───────┐                              │
  │  │ Containers   │  │ File Shares  │                              │
  │  └──────┬───────┘  └──────┬───────┘                              │
  │         │                 │                                      │
  │  ┌──────▼───────┐  ┌──────▼───────┐                              │
  │  │ Blobs        │  │ Directories/ │                              │
  │  │ (Files)      │  │ Files        │                              │
  │  └──────────────┘  └──────────────┘                              │
  └──────────────────────────────────────────────────────────────────┘
```

#### Creating a Storage Account

```bash
# Create a standard general-purpose v2 storage account with ZRS
az storage account create \
  --name "az104storageapp123" \
  --resource-group "az104-lab-rg" \
  --location "eastus" \
  --sku "Standard_ZRS" \
  --kind "StorageV2"
```

### 3.3 Redundancy Choices

Data is always replicated to ensure durability. The choice is *where* it is replicated.

| Option | Name | What it protects against | Copies | Best for |
|--------|------|--------------------------|--------|----------|
| LRS | Locally Redundant | Rack/drive failure | 3 in one zone | Dev/test, easily reproducible data |
| ZRS | Zone Redundant | Datacenter failure | 3 across 3 zones | Production apps needing high availability within a region |
| GRS | Geo-Redundant | Regional disaster | 3 local + 3 in paired region | Backups, disaster recovery |
| RA-GRS | Read-Access GRS | Regional disaster | 3 local + 3 in paired region | High availability with read access during regional outage |

### 3.4 Storage Tiers (Blob)

Tiers control the balance between storage cost and access cost.

| Tier | Best for | Storage Cost | Access Cost |
|------|----------|--------------|-------------|
| Hot | Frequently accessed data | Highest | Lowest |
| Cool | Infrequently accessed (>= 30 days) | Lower | Higher |
| Cold | Rarely accessed (>= 90 days) | Very Low | Very High |
| Archive | Rarely accessed (>= 180 days) | Lowest | Highest (and hours to retrieve) |

> **Production Gotcha: The Archive Rehydration Trap**
>
> **The Problem:** You put a 500GB database backup in Archive tier to save money. A disaster happens, and you need the backup *now*.
>
> **Why You'll Hit This:** Archive tier is offline. Reading data requires "rehydrating" it to a Hot or Cool tier, which takes up to 15 hours.
>
> **The Fix:** Only use Archive for data you are legally required to keep but hope to never read. Use Cool or Cold for disaster recovery backups you might actually need quickly.

### 3.5 Lifecycle Management

Automate moving data between tiers to save money.

```bash
# Example JSON policy (saved as policy.json)
# {
#   "rules": [
#     {
#       "enabled": true,
#       "name": "MoveToCoolThenArchive",
#       "type": "Lifecycle",
#       "definition": {
#         "actions": {
#           "baseBlob": {
#             "tierToCool": {"daysAfterModificationGreaterThan": 30},
#             "tierToArchive": {"daysAfterModificationGreaterThan": 90},
#             "delete": {"daysAfterModificationGreaterThan": 2555}
#           }
#         },
#         "filters": {"blobTypes": ["blockBlob"]}
#       }
#     }
#   ]
# }

# Apply the lifecycle management policy
az storage account management-policy create \
  --account-name "az104storageapp123" \
  --resource-group "az104-lab-rg" \
  --policy @policy.json
```

### 3.6 Access Control for Storage

This is highly tested. You must know when to use which authentication method.

| Method | What it is | Best for |
|--------|------------|----------|
| Entra ID (RBAC) | Identity-based access (e.g., "Storage Blob Data Contributor") | Human access, Managed Identities |
| Storage Account Keys | Two master keys giving root access to the entire account | Legacy apps, emergency admin access (avoid if possible) |
| Shared Access Signature (SAS) | Time-bound, scoped URI granting specific permissions | Providing a client direct download/upload access |
| Stored Access Policy | Server-side policy that groups SAS tokens | Scenarios where you might need to revoke a SAS before it expires |

#### Generating a SAS Token

```bash
# Generate a SAS token for a specific blob, valid for 1 hour, read-only
end=`date -u -d "1 hour" '+%Y-%m-%dT%H:%MZ'`

az storage blob generate-sas \
  --account-name "az104storageapp123" \
  --container-name "images" \
  --name "logo.png" \
  --permissions r \
  --expiry $end \
  --https-only
```

> **Exam Trap:** SAS vs Keys. If a question mentions "delegated access", "limited time", "specific permissions", or "external clients", the answer is almost always **SAS**.

### 3.7 Storage Network Security (Firewall & Private Endpoints)

By default, storage accounts are accessible from all networks via their public endpoint.

```
  ┌────────────────────────────────────────────────────────────┐
  │                  Storage Network Access                    │
  │                                                            │
  │  1. Public Endpoint (All Networks)                         │
  │     Internet ────► [Storage Account]                       │
  │                                                            │
  │  2. Service Endpoint (Public IP, but restricted path)      │
  │     VNet Subnet A ────► [Storage Account]                  │
  │     (Traffic stays on Azure backbone, but uses public IP)  │
  │                                                            │
  │  3. Private Endpoint (Private IP inside VNet)              │
  │     VNet Subnet A ────► [10.0.1.5 (Private Link)]          │
  │     (No public IP required, traffic is entirely private)   │
  └────────────────────────────────────────────────────────────┘
```

#### Configuring Network Rules

```bash
# Set default action to Deny (blocks all public access)
az storage account update \
  --name "az104storageapp123" \
  --resource-group "az104-lab-rg" \
  --default-action Deny

# Allow access from a specific IP (e.g., corporate VPN)
az storage account network-rule add \
  --account-name "az104storageapp123" \
  --resource-group "az104-lab-rg" \
  --ip-address "203.0.113.5"

# Allow access from a specific VNet subnet (Service Endpoint)
# Note: The subnet must have the Microsoft.Storage service endpoint enabled first
az storage account network-rule add \
  --account-name "az104storageapp123" \
  --resource-group "az104-lab-rg" \
  --vnet-name "az104-vnet" \
  --subnet "app-subnet"
```

### 3.8 Azure Files vs Blob Storage

| Feature | Azure Files | Blob Storage |
|---------|-------------|--------------|
| Access Protocol | SMB 3.0 / REST | REST |
| Folder Structure | Hierarchical | Flat (virtual folders) |
| Typical Use | Lift-and-shift file servers, shared app config | Massive unstructured data, backups, images |
| Authentication | Entra ID (for SMB), Storage Keys | Entra ID, SAS, Storage Keys |

### 3.9 Data Protection (Soft Delete & Immutability)

| Feature | What it does | Typical exam cue |
|---------|--------------|-------------------|
| Soft Delete (Blob/Container) | Retains deleted items for a set number of days before permanent deletion. | "Recover accidentally deleted blobs" |
| Point-in-time Restore | Restores entire container state to a past moment (requires soft delete + versioning + change feed). | "Roll back from a ransomware attack" |
| Immutability Policy (WORM) | Write-Once, Read-Many. Prevents modification or deletion for a defined retention period. | "Legal compliance", "SEC regulations" |

### 3.10 Storage Data Transfer Tools

| Tool | Best for | Key Feature |
|------|----------|-------------|
| AzCopy | Command-line bulk transfer | Highly optimized, syncs directories |
| Storage Explorer | GUI management | Browsing, simple uploads/downloads |
| Azure Data Box | Petabyte-scale offline transfer | Microsoft ships you a physical hard drive/appliance |
| Azure File Sync | Hybrid file sharing | Caches on-prem Windows Server files in Azure Files |


## 4. Compute

### 4.1 Why Compute Matters

As an Azure administrator, you **will** encounter:
- Legacy applications that require a specific, older OS version and full administrative control.
- Modern web apps that need to scale automatically based on traffic without you managing the underlying servers.
- Containerized microservices that need to run quickly without orchestration overhead.
- Spikes in traffic during events that require your application to scale out rapidly.

Compute in Azure is about choosing the right level of abstraction for the workload: do you want to manage the OS (IaaS), or just the application code (PaaS/Serverless)?

### 4.2 Compute Decision Model

The most important skill is knowing which compute service to choose.

```
  ┌────────────────────────────────────────────────────────────┐
  │                   Compute Decision Tree                    │
  │                                                            │
  │  Need full OS control? ── Yes ──► Virtual Machine          │
  │         │                                                  │
  │         No                                                 │
  │         ▼                                                  │
  │  Need multiple identical VMs for scale/HA?                 │
  │         ├── Yes ──► Virtual Machine Scale Set (VMSS)       │
  │         │                                                  │
  │         No                                                 │
  │         ▼                                                  │
  │  Need managed web hosting (PaaS)?                          │
  │         ├── Yes ──► App Service                            │
  │         │                                                  │
  │         No                                                 │
  │         ▼                                                  │
  │  Running containers?                                       │
  │         ├── Fast, simple run ──► Container Instances (ACI) │
  │         ├── Event-driven microservices ──► Container Apps  │
  │         └── Full orchestration ──► Kubernetes Service (AKS)│
  └────────────────────────────────────────────────────────────┘
```

### 4.3 Virtual Machines (IaaS)

When you create a VM, you are actually creating multiple interconnected resources.

```
  ┌────────────────────────────────────────────────────────────┐
  │                    Virtual Machine Anatomy                 │
  │                                                            │
  │  ┌───────────────┐                                         │
  │  │ Virtual       │────► [OS Disk] (Managed Disk)           │
  │  │ Machine       │                                         │
  │  │ (Compute/RAM) │────► [Data Disk] (Optional)             │
  │  └───────┬───────┘                                         │
  │          │                                                 │
  │  ┌───────▼───────┐      ┌─────────────┐    ┌─────────────┐ │
  │  │ Network       │────► │ Public IP   │ or │ Load        │ │
  │  │ Interface     │      │ (Optional)  │    │ Balancer    │ │
  │  │ (NIC)         │      └─────────────┘    └─────────────┘ │
  │  └───────┬───────┘                                         │
  │          │                                                 │
  │  ┌───────▼───────┐                                         │
  │  │ Subnet / VNet │                                         │
  │  └───────────────┘                                         │
  └────────────────────────────────────────────────────────────┘
```

#### Deploying a VM via CLI

```bash
# Create a Resource Group and VNet first (best practice)
az group create --name az104-compute-rg --location eastus
az network vnet create --resource-group az104-compute-rg --name myVNet --subnet-name mySubnet

# Create the VM (this automatically creates the NIC, OS Disk, and connects to the subnet)
az vm create \
  --resource-group az104-compute-rg \
  --name myVM \
  --image Ubuntu2204 \
  --vnet-name myVNet \
  --subnet mySubnet \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard

# Stop (Deallocate) the VM
az vm deallocate --resource-group az104-compute-rg --name myVM
```

> **Exam Trap: Stopped vs. Deallocated**
>
> **Stopped:** The OS is shut down, but the VM hardware is still allocated to you. **You are still paying for the compute hours.**
>
> **Deallocated:** The VM is released from the Azure host hardware. **Compute billing stops.** (You still pay for the disks). Always use `deallocate` when you want to stop paying.

### 4.4 VM Disks

Azure VMs use Managed Disks. You don't need to manage the underlying storage accounts for them.

| Disk Type | Best For | Performance |
|-----------|----------|-------------|
| Ultra Disk | SAP HANA, top tier databases | Sub-millisecond latency, massive IOPS |
| Premium SSD v2 | High performance databases (SQL Server, Oracle) | Consistently high IOPS |
| Premium SSD | Production workloads, OS disks | High IOPS, single-digit ms latency |
| Standard SSD | Web servers, dev/test | Reliable, cost-effective |
| Standard HDD | Backups, non-critical data | Lowest cost, spinning disk |

#### Disk Operations

```bash
# Attach a new data disk to an existing VM
az vm disk attach \
  --resource-group az104-compute-rg \
  --vm-name myVM \
  --name myDataDisk \
  --new \
  --size-gb 128 \
  --sku Premium_LRS

# Create a snapshot of a disk (for backup before a risky change)
az snapshot create \
  --resource-group az104-compute-rg \
  --name myOSDiskSnapshot \
  --source myVM_OsDisk_1_123456
```

### 4.5 VM Availability

Hardware fails. Azure requires you to plan for it.

| Feature | Protects Against | SLA | Concept |
|---------|------------------|-----|---------|
| Single VM (Premium SSD) | - | 99.9% | Basic SLA |
| Availability Set | Rack/Server failure | 99.95% | Places VMs in different fault domains (racks) within the *same* datacenter. |
| Availability Zone | Datacenter failure | 99.99% | Places VMs in physically separate datacenters within the *same* region. |

### 4.6 Virtual Machine Scale Sets (VMSS)

VMSS allows you to create and manage a group of load-balanced, identical VMs. The number of VM instances can automatically increase or decrease in response to demand or a defined schedule.

*   **Identical VMs:** Built from the same base image.
*   **Auto-scaling:** Scale out (add instances) when CPU is high. Scale in (remove instances) when idle.
*   **Load Balancing:** Integrated closely with Azure Load Balancer or Application Gateway.

```bash
# Create a VMSS with auto-scaling enabled
az vmss create \
  --resource-group az104-compute-rg \
  --name myScaleSet \
  --image Ubuntu2204 \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys \
  --instance-count 2
```

### 4.7 App Service (PaaS)

App Service is a managed platform for hosting web apps, REST APIs, and mobile backends. You deploy code (or containers), Azure manages the infrastructure.

#### App Service Plan

The App Service Plan is the *compute* behind the App Service. It defines the region, instance size, and features. You pay for the Plan, not the individual apps within it.

| Plan Tier | Key Features | Best For |
|-----------|--------------|----------|
| Free / Shared | - | Learning, small experiments (No SLA) |
| Basic | Custom domains, SSL | Dev/test |
| Standard | Auto-scale, Daily backups, Slots | Production web apps |
| Premium | High performance, VNet integration | High-traffic, secure enterprise apps |
| Isolated | Dedicated environment (ASE) | Maximum security and isolation |

#### Deployment Slots

Slots are live apps with their own hostnames. You can deploy to a staging slot, test it, and then **swap** it with the production slot with zero downtime.

```bash
# Create an App Service Plan
az appservice plan create --name myAppPlan --resource-group az104-compute-rg --sku S1

# Create the Web App
az webapp create --name myUniqueWebApp123 --resource-group az104-compute-rg --plan myAppPlan

# Create a staging deployment slot
az webapp deployment slot create --name myUniqueWebApp123 --resource-group az104-compute-rg --slot staging

# Swap staging into production
az webapp deployment slot swap --name myUniqueWebApp123 --resource-group az104-compute-rg --slot staging --target-slot production
```

### 4.8 Containers (ACI & ACR)

| Service | Purpose |
|---------|---------|
| Azure Container Registry (ACR) | A private Docker registry in Azure to store your container images securely. |
| Azure Container Instances (ACI) | The fastest way to run a container in Azure, without having to provision VMs or adopt a higher-level service. |

```bash
# Create an ACR
az acr create --resource-group az104-compute-rg --name myprivateacr123 --sku Basic

# Run a container in ACI
az container create \
  --resource-group az104-compute-rg \
  --name mycontainer \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --dns-name-label myaci-demo \
  --ports 80
```

### 4.9 Infrastructure as Code (ARM & Bicep)

ARM templates (JSON) and Bicep (cleaner syntax) are how you deploy infrastructure declaratively. The exam expects you to understand the structure of an ARM template.

| Section | Purpose |
|---------|---------|
| `parameters` | Values provided at deployment time (e.g., VM name, admin password). |
| `variables` | Values constructed within the template (e.g., combining strings for a unique name). |
| `resources` | The actual Azure resources to be deployed (e.g., `Microsoft.Compute/virtualMachines`). |
| `outputs` | Values returned after deployment completes (e.g., the new VM's public IP address). |

```bash
# Deploy an ARM template
az deployment group create \
  --resource-group az104-compute-rg \
  --template-file azuredeploy.json \
  --parameters adminUsername=azureuser adminPassword=SecretPassword123!
```

---

## 5. Virtual Networking

### 5.1 Why Networking Matters

As an Azure administrator, you **will** encounter:
- Applications that need to communicate securely between subnets without being exposed to the internet.
- Debugging a connection that times out — is it DNS? NSG? Route table? Health probe?
- Connecting on-premises networks to Azure for hybrid workloads.
- Securing admin access to VMs without exposing SSH/RDP ports to the internet.

Networking in Azure is about **how traffic reaches the right place safely**, and how you troubleshoot when it doesn't.

### 5.2 Virtual Network Mental Model

```
  ┌───────────────────────────────────────────────────────────────┐
  │                      Virtual Network (VNet)                    │
  │                    Address Space: 10.0.0.0/16                  │
  │                                                                │
  │  ┌───────────────────┐  ┌───────────────────┐                  │
  │  │  Subnet A         │  │  Subnet B         │                  │
  │  │  10.0.1.0/24      │  │  10.0.2.0/24      │                  │
  │  │  ┌─────┐  ┌─────┐ │  │  ┌─────┐  ┌─────┐ │                  │
  │  │  │ VM1 │  │ VM2 │ │  │  │ DB  │  │ App │ │                  │
  │  │  └─────┘  └─────┘ │  │  └─────┘  └─────┘ │                  │
  │  │  NSG: allow 80,443│  │  NSG: allow 1433   │                  │
  │  └───────────────────┘  └───────────────────┘                  │
  │                                                                │
  │  ┌───────────────────┐  ┌───────────────────┐                  │
  │  │  AzureBastionSub  │  │  GatewaySubnet    │                  │
  │  │  10.0.3.0/27      │  │  10.0.4.0/27      │                  │
  │  │  (Bastion host)   │  │  (VPN/ER Gateway) │                  │
  │  └───────────────────┘  └───────────────────┘                  │
  └───────────────────────────────────────────────────────────────┘
```

**Reserved subnet names:**
- `AzureBastionSubnet` — must be named exactly this, minimum /27
- `GatewaySubnet` — must be named exactly this for VPN/ExpressRoute gateways

#### Creating a VNet

```bash
# Create a VNet with two subnets
az network vnet create \
  --resource-group az104-lab-rg \
  --name az104-vnet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name app-subnet \
  --subnet-prefix 10.0.1.0/24

# Add a second subnet
az network vnet subnet create \
  --resource-group az104-lab-rg \
  --vnet-name az104-vnet \
  --name data-subnet \
  --address-prefix 10.0.2.0/24
```

### 5.3 NSGs and ASGs

#### NSG Rule Evaluation Flow

NSG rules are evaluated by **priority number** (lower number = higher priority). The first matching rule wins.

```
  ┌───────────────────────────────────────────────────────────────┐
  │                   NSG RULE EVALUATION                         │
  │                                                               │
  │  Inbound packet arrives at NIC                                │
  │        │                                                      │
  │        ▼                                                      │
  │  ┌─────────────────────────┐                                  │
  │  │  Subnet NSG exists?     │                                  │
  │  │  ├── Yes: Evaluate rules│ (lowest priority # first)       │
  │  │  │   ├── Match: Allow ──► Continue to NIC NSG check       │
  │  │  │   ├── Match: Deny ───► DROP packet                     │
  │  │  │   └── No match: Apply default deny                     │
  │  │  └── No: Continue       │                                  │
  │  └─────────┬───────────────┘                                  │
  │            ▼                                                  │
  │  ┌─────────────────────────┐                                  │
  │  │  NIC NSG exists?        │                                  │
  │  │  ├── Yes: Evaluate rules│ (lowest priority # first)       │
  │  │  │   ├── Match: Allow ──► DELIVER to VM                   │
  │  │  │   ├── Match: Deny ───► DROP packet                     │
  │  │  │   └── No match: Apply default deny                     │
  │  │  └── No: DELIVER to VM  │                                  │
  │  └─────────────────────────┘                                  │
  │                                                               │
  │  Outbound: Reverse order (NIC NSG first, then Subnet NSG)    │
  └───────────────────────────────────────────────────────────────┘
```

**Default rules (cannot be deleted, always lowest priority):**

| Priority | Name | Direction | Action |
|----------|------|-----------|--------|
| 65000 | AllowVnetInBound | Inbound | Allow VNet-to-VNet |
| 65001 | AllowAzureLoadBalancerInBound | Inbound | Allow health probes |
| 65500 | DenyAllInBound | Inbound | Deny everything else |

#### Creating NSG Rules

```bash
# Create an NSG
az network nsg create \
  --resource-group az104-lab-rg \
  --name web-nsg

# Allow HTTP inbound
az network nsg rule create \
  --resource-group az104-lab-rg \
  --nsg-name web-nsg \
  --name AllowHTTP \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 80 443

# Deny all other inbound (explicit — for clarity, default already does this)
az network nsg rule create \
  --resource-group az104-lab-rg \
  --nsg-name web-nsg \
  --name DenyAllInbound \
  --priority 4000 \
  --direction Inbound \
  --access Deny \
  --protocol '*' \
  --destination-port-ranges '*'

# Associate NSG with a subnet
az network vnet subnet update \
  --resource-group az104-lab-rg \
  --vnet-name az104-vnet \
  --name app-subnet \
  --network-security-group web-nsg

# View effective security rules for a NIC
az network nic list-effective-nsg \
  --resource-group az104-lab-rg \
  --name myVM-nic
```

**ASGs (Application Security Groups):** Group NICs by application role instead of IP address. Instead of writing rules for `10.0.1.4`, write rules for `web-servers` ASG.

### 5.4 VNet Peering

VNet peering connects two VNets so resources can communicate using private IP addresses. Traffic does not traverse the public internet.

```
  ┌──────────────┐         Peering         ┌──────────────┐
  │  VNet A      │◄────────────────────────►│  VNet B      │
  │  10.0.0.0/16 │   (private, low latency) │  10.1.0.0/16 │
  └──────────────┘                          └──────────────┘
```

**Key facts for the exam:**
- Peering is **not transitive** — if A peers with B and B peers with C, A cannot reach C unless A also peers with C.
- Address spaces **must not overlap**.
- Peering can be within a region (VNet peering) or cross-region (Global VNet peering).

```bash
# Create peering from VNet A to VNet B
az network vnet peering create \
  --resource-group az104-lab-rg \
  --name AtoB \
  --vnet-name vnetA \
  --remote-vnet /subscriptions/<sub-id>/resourceGroups/az104-lab-rg/providers/Microsoft.Network/virtualNetworks/vnetB \
  --allow-vnet-access

# Peering must be created from BOTH sides
az network vnet peering create \
  --resource-group az104-lab-rg \
  --name BtoA \
  --vnet-name vnetB \
  --remote-vnet /subscriptions/<sub-id>/resourceGroups/az104-lab-rg/providers/Microsoft.Network/virtualNetworks/vnetA \
  --allow-vnet-access
```

### 5.5 User-Defined Routes (UDRs)

Azure has default system routes that handle most traffic. UDRs override those defaults when you need traffic to flow through a firewall or network virtual appliance (NVA).

```bash
# Create a route table
az network route-table create \
  --resource-group az104-lab-rg \
  --name custom-routes

# Add a route that forces all internet-bound traffic through a firewall NVA
az network route-table route create \
  --resource-group az104-lab-rg \
  --route-table-name custom-routes \
  --name force-through-firewall \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address 10.0.3.4

# Associate route table with a subnet
az network vnet subnet update \
  --resource-group az104-lab-rg \
  --vnet-name az104-vnet \
  --name app-subnet \
  --route-table custom-routes
```

### 5.6 Azure Load Balancer (Layer 4)

Azure Load Balancer distributes traffic at the **transport layer** (TCP/UDP). It does not understand HTTP.

```
  ┌─────────────────────────────────────────────────────────────┐
  │                Azure Load Balancer (L4)                      │
  │                                                              │
  │  ┌──────────────┐    ┌──────────────────┐    ┌────────────┐  │
  │  │ Frontend IP  │───►│ Load Balancing   │───►│ Backend    │  │
  │  │ (Public or   │    │ Rule             │    │ Pool       │  │
  │  │  Private)    │    │ (Port 80 → 80)   │    │ ┌────────┐ │  │
  │  └──────────────┘    └──────────────────┘    │ │ VM 1   │ │  │
  │                                              │ │ VM 2   │ │  │
  │                      ┌──────────────────┐    │ │ VM 3   │ │  │
  │                      │ Health Probe     │    │ └────────┘ │  │
  │                      │ (TCP 80 every    │───►│ (removes   │  │
  │                      │  15 seconds)     │    │  unhealthy)│  │
  │                      └──────────────────┘    └────────────┘  │
  └─────────────────────────────────────────────────────────────┘
```

| Type | Use case |
|------|----------|
| Public LB | Distribute internet traffic to VMs |
| Internal LB | Distribute traffic between tiers (web → app → db) |

```bash
# Create a public load balancer
az network lb create \
  --resource-group az104-lab-rg \
  --name myLoadBalancer \
  --sku Standard \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool \
  --public-ip-address myPublicIP

# Add a health probe
az network lb probe create \
  --resource-group az104-lab-rg \
  --lb-name myLoadBalancer \
  --name myHealthProbe \
  --protocol tcp \
  --port 80

# Add a load balancing rule
az network lb rule create \
  --resource-group az104-lab-rg \
  --lb-name myLoadBalancer \
  --name myLBRule \
  --protocol tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool \
  --probe-name myHealthProbe
```

### 5.7 Application Gateway (Layer 7)

Application Gateway is a **web traffic load balancer** that operates at HTTP/HTTPS level. Unlike the Layer 4 Load Balancer, it understands URLs, headers, and cookies.

```
  ┌──────────────────────────────────────────────────────────────┐
  │            Application Gateway vs Load Balancer               │
  │                                                               │
  │  Load Balancer (L4)        Application Gateway (L7)           │
  │  ┌────────────────────┐    ┌─────────────────────────────┐    │
  │  │ TCP/UDP only        │    │ HTTP/HTTPS aware             │    │
  │  │ Port-based routing  │    │ URL path-based routing       │    │
  │  │ No SSL termination  │    │ SSL termination ✅           │    │
  │  │ No WAF              │    │ Web Application Firewall ✅  │    │
  │  │ No cookie affinity  │    │ Cookie-based affinity ✅     │    │
  │  │ No URL rewrite      │    │ URL rewrite/redirect ✅      │    │
  │  └────────────────────┘    └─────────────────────────────┘    │
  │                                                               │
  │  Exam cue:                                                    │
  │  "Route /api/* to backend A, /images/* to backend B"          │
  │  → Application Gateway                                       │
  │  "Distribute TCP traffic to a pool of VMs"                    │
  │  → Load Balancer                                              │
  └──────────────────────────────────────────────────────────────┘
```

### 5.8 Azure DNS

| Service | Purpose |
|---------|---------|
| Azure DNS (Public) | Host public DNS zones (e.g., `example.com`) and manage records |
| Private DNS Zone | Name resolution inside VNets (e.g., `privatelink.blob.core.windows.net`) |

```bash
# Create a public DNS zone
az network dns zone create \
  --resource-group az104-lab-rg \
  --name example.com

# Add an A record
az network dns record-set a add-record \
  --resource-group az104-lab-rg \
  --zone-name example.com \
  --record-set-name www \
  --ipv4-address 20.30.40.50

# Create a private DNS zone (for Private Endpoints)
az network private-dns zone create \
  --resource-group az104-lab-rg \
  --name privatelink.blob.core.windows.net

# Link private DNS zone to a VNet
az network private-dns link vnet create \
  --resource-group az104-lab-rg \
  --zone-name privatelink.blob.core.windows.net \
  --name myDNSLink \
  --virtual-network az104-vnet \
  --registration-enabled false
```

### 5.9 Azure Bastion — Secure Admin Access

Bastion provides secure RDP/SSH to VMs without exposing public IP addresses. It is deployed inside the `AzureBastionSubnet` and accessed through the Azure Portal.

| Feature | Why it matters |
|---------|----------------|
| No public IP needed on VMs | Reduces attack surface |
| Browser-based RDP/SSH | No client software needed |
| NSG rules simplified | No SSH/RDP port in VM NSG |
| Audit logging | All sessions are logged |

### 5.10 Service Endpoints vs Private Endpoints

| Feature | Service Endpoint | Private Endpoint |
|---------|-----------------|-----------------|
| How it works | Optimized route from VNet to PaaS public endpoint | Private IP address in your VNet |
| DNS impact | None | Requires Private DNS zone |
| Data path | Azure backbone (but still public IP) | Fully private network path |
| Best for | Simple "restrict to my VNet" scenarios | Zero-trust, compliance-heavy environments |

> **Networking Gotcha: Private Endpoint DNS Failures**
>
> **The Problem:** You create a private endpoint for a storage account. Portal shows it's connected. But `nslookup mystorageaccount.blob.core.windows.net` still resolves to a public IP, and your app times out.
>
> **Why You'll Hit This:** Private endpoints require DNS integration. Without a properly linked Private DNS Zone, clients resolve the public IP which is blocked by the firewall rule you set.
>
> **The Fix:** Create a Private DNS Zone (`privatelink.blob.core.windows.net`), link it to your VNet, and the private endpoint will auto-register its private IP. Verify with `nslookup` — it should return `10.0.x.x`, not a public IP.

### 5.11 VPN Gateway — Hybrid Connectivity

VPN Gateway connects on-premises networks to Azure over an encrypted tunnel (IPsec/IKE).

```
  ┌──────────────────┐         IPsec Tunnel         ┌──────────────────┐
  │  On-Premises     │◄────────────────────────────►│  Azure VNet      │
  │  Network         │       (over internet)        │                  │
  │  192.168.0.0/16  │                              │  10.0.0.0/16     │
  │                  │                              │                  │
  │  [Local Network  │                              │  [VPN Gateway    │
  │   Gateway]       │                              │   in GatewaySubnet]│
  └──────────────────┘                              └──────────────────┘
```

| Connection Type | Use case |
|----------------|----------|
| Site-to-Site (S2S) | On-premises datacenter to Azure — persistent tunnel |
| Point-to-Site (P2S) | Individual client (laptop) to Azure — remote workers |
| VNet-to-VNet | Connect VNets across regions via gateway (alternative to peering) |

### 5.12 ExpressRoute — Dedicated Private Connection

ExpressRoute provides a **private, dedicated connection** to Azure that does not traverse the public internet. It offers higher bandwidth, lower latency, and more reliability than VPN.

| Feature | VPN Gateway | ExpressRoute |
|---------|-------------|-------------|
| Path | Over public internet | Private connection via provider |
| Encryption | IPsec (encrypted) | Not encrypted by default (private path) |
| Bandwidth | Up to ~1.25 Gbps | Up to 100 Gbps |
| Latency | Variable | Predictable, low |
| Best for | Dev/test, small-medium hybrid | Enterprise hybrid, large data transfers |

### 5.13 Network Watcher — Diagnostics and Troubleshooting

Network Watcher is your troubleshooting toolkit for Azure networking.

| Tool | What it does | When to use |
|------|-------------|-------------|
| IP Flow Verify | Checks if a packet is allowed or denied by NSG | "Why can't VM A reach VM B on port 443?" |
| Next Hop | Shows where a packet will be routed | "Is traffic going through my NVA or directly?" |
| Connection Troubleshoot | End-to-end connectivity check between two endpoints | "Is the connection timing out at the NSG, route, or destination level?" |
| NSG Flow Logs | Records all traffic allowed/denied by NSGs | Security auditing, traffic analysis |
| Packet Capture | Captures packets on a VM NIC | Deep network debugging |

```bash
# Check if traffic from VM to destination is allowed
az network watcher test-ip-flow \
  --resource-group az104-lab-rg \
  --vm myVM \
  --direction Inbound \
  --protocol TCP \
  --local 10.0.1.4:80 \
  --remote 203.0.113.5:12345

# Check next hop for a destination
az network watcher show-next-hop \
  --resource-group az104-lab-rg \
  --vm myVM \
  --source-ip 10.0.1.4 \
  --dest-ip 10.0.2.4
```

### 5.14 Networking Debug Order

When networking fails, check in this order:

1. **DNS resolution** — does the name resolve to the right IP?
2. **IP routing** — is the packet going where you expect? (Next Hop)
3. **NSG or firewall rules** — is the port/protocol allowed? (IP Flow Verify)
4. **Load balancer health probes** — is the backend marked healthy?
5. **Private endpoint / service endpoint** — is the DNS correctly pointing to the private IP?

> **Networking Gotcha: Health Probe ≠ Application Health**
>
> **The Problem:** Load balancer says all backends are healthy, but users see 500 errors.
>
> **Why You'll Hit This:** The health probe checks TCP port 80. The app listens on port 80 but its database connection is broken. TCP handshake succeeds, probe passes, but application fails.
>
> **The Fix:** Use HTTP health probes with a path (e.g., `/health`) that checks actual dependencies, not just port availability.

---

## 6. Monitoring and Maintenance

### 6.1 Why Monitoring Matters

As an Azure administrator, you **will** encounter:
- An alert at 3 AM that a production VM's CPU is at 100% — you need to know where to look and how to query the data.
- A compliance audit asking "show me all resource deletions in the last 30 days."
- A cost spike that needs root cause analysis — which resource, which team, which operation?
- A disaster recovery scenario where you need to restore a VM or database to a point in time.

Monitoring is about **what happened, where, and why** — and how to get notified before users do.

### 6.2 Monitoring Pipeline Mental Model

```
  ┌───────────────────────────────────────────────────────────────────┐
  │                   AZURE MONITORING PIPELINE                       │
  │                                                                   │
  │  ┌──────────────────┐                                             │
  │  │  Azure Resource   │                                             │
  │  │  (VM, Storage,    │                                             │
  │  │   App Service)    │                                             │
  │  └────────┬─────────┘                                             │
  │           │  emits                                                │
  │           ▼                                                       │
  │  ┌─────────────────────────────────────────────────┐              │
  │  │  Telemetry Types                                 │              │
  │  │  ├── Metrics ────► numeric time series (CPU, %)  │              │
  │  │  ├── Logs ───────► structured text (errors, ops) │              │
  │  │  └── Activity Log► control-plane audit trail     │              │
  │  └────────┬────────────────────────────────────────┘              │
  │           │  routed via Diagnostic Settings                       │
  │           ▼                                                       │
  │  ┌─────────────────────────────────────────────────────────────┐  │
  │  │  Destinations                                                │  │
  │  │  ├── Log Analytics Workspace ──► KQL queries, dashboards    │  │
  │  │  ├── Storage Account ──────────► long-term archival          │  │
  │  │  └── Event Hub ────────────────► stream to SIEM / 3rd party │  │
  │  └────────┬────────────────────────────────────────────────────┘  │
  │           ▼                                                       │
  │  ┌─────────────────────────────────────────────────┐              │
  │  │  Alert Rule ──► Action Group ──► Notification   │              │
  │  │  (condition)    (who + how)      (email, SMS,   │              │
  │  │                                   webhook, ITSM)│              │
  │  └─────────────────────────────────────────────────┘              │
  └───────────────────────────────────────────────────────────────────┘
```

### 6.3 Metrics vs Logs vs Activity Log

| Telemetry | What it is | Best for | Example |
|-----------|-----------|----------|---------|
| Metrics | Numeric data collected at intervals | Real-time dashboards, threshold alerts | CPU %, Disk IOPS, HTTP request count |
| Logs | Structured text records with rich detail | Deep investigation, correlation | App errors, diagnostic traces |
| Activity Log | Record of control-plane operations | Audit trail | Who deleted resource X? When was policy Y assigned? |

### 6.4 Diagnostic Settings

Diagnostic settings **route** platform telemetry from a resource to a destination. Without them, data stays inside the resource.

```bash
# Enable diagnostic settings to send to Log Analytics
az monitor diagnostic-settings create \
  --resource /subscriptions/<sub-id>/resourceGroups/az104-lab-rg/providers/Microsoft.Compute/virtualMachines/myVM \
  --name send-to-law \
  --workspace /subscriptions/<sub-id>/resourceGroups/az104-lab-rg/providers/Microsoft.OperationalInsights/workspaces/az104-law \
  --metrics '[{"category":"AllMetrics","enabled":true}]' \
  --logs '[{"category":"Administrative","enabled":true}]'
```

> **Monitoring Gotcha: "Where Is My Data?"**
>
> **The Problem:** You create a Log Analytics workspace and expect to see VM logs. Nothing appears.
>
> **Why You'll Hit This:** Creating a workspace does not automatically collect data. You must configure **diagnostic settings** on each resource.
>
> **The Fix:** Always verify diagnostic settings on the source resource. Most monitoring problems are ingestion problems, not query problems.

### 6.5 Log Analytics and KQL

```bash
# Create a Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group az104-lab-rg \
  --workspace-name az104-law \
  --location eastus
```

#### KQL — Essential Query Patterns

```
// Show recent heartbeat signals from VMs
Heartbeat
| where TimeGenerated > ago(1h)
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| order by LastHeartbeat asc

// Count events by severity in the last 24 hours
Event
| where TimeGenerated > ago(24h)
| summarize Count = count() by EventLevelName
| order by Count desc

// Find all failed sign-ins in the last 7 days
SigninLogs
| where TimeGenerated > ago(7d)
| where ResultType != "0"
| summarize FailedAttempts = count() by UserPrincipalName, IPAddress
| order by FailedAttempts desc

// Track CPU trends for a specific VM
Perf
| where TimeGenerated > ago(4h)
| where Computer == "myVM"
| where CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by bin(TimeGenerated, 5m)
| render timechart
```

| KQL Operator | What it does |
|-------------|-------------|
| `where` | Filter rows |
| `summarize` | Aggregate (count, avg, sum, max) |
| `project` | Select/rename columns |
| `order by` | Sort results |
| `bin()` | Group time into buckets |
| `ago()` | Relative time |
| `render` | Visualize as chart |

### 6.6 Alerts and Action Groups

```bash
# Create an action group
az monitor action-group create \
  --resource-group az104-lab-rg \
  --name ops-team-alerts \
  --short-name opsalert \
  --action email ops-email ops@company.com

# Create a metric alert (CPU > 80% for 5 minutes)
az monitor metrics alert create \
  --name HighCPUAlert \
  --resource-group az104-lab-rg \
  --scopes /subscriptions/<sub-id>/resourceGroups/az104-lab-rg/providers/Microsoft.Compute/virtualMachines/myVM \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action ops-team-alerts \
  --severity 2 \
  --description "CPU exceeds 80% sustained for 5 minutes"
```

| Alert type | Trigger | Best for |
|-----------|---------|----------|
| Metric alert | Numeric threshold crossed | CPU, memory, disk, request count |
| Log alert | KQL query returns results | Custom conditions, complex logic |
| Activity log alert | Control-plane operation occurs | Resource deleted, policy changed |
| Service health alert | Azure platform issue | Regional outages, planned maintenance |

### 6.7 Azure Backup

Azure Backup provides zero-infrastructure backup management via **Recovery Services Vaults**.

```
  ┌───────────────────────────────────────────────────────────────┐
  │                    AZURE BACKUP ARCHITECTURE                   │
  │                                                                │
  │  ┌────────────┐     ┌──────────────────────┐                   │
  │  │ Source      │────►│ Recovery Services    │                   │
  │  │ (VM, SQL,   │     │ Vault                │                   │
  │  │  File Share)│     │ • Backup policies    │                   │
  │  └────────────┘     │ • Retention rules    │                   │
  │                     │ • Geo-replication     │                   │
  │                     │ • Soft delete (14d)   │                   │
  │                     └──────────┬───────────┘                   │
  │                                │                               │
  │                     ┌──────────▼───────────┐                   │
  │                     │ Recovery Points       │                   │
  │                     │ Daily/Weekly/Monthly  │                   │
  │                     │ Restore VM, disks,    │                   │
  │                     │ or individual files   │                   │
  │                     └──────────────────────┘                   │
  └───────────────────────────────────────────────────────────────┘
```

```bash
# Create a Recovery Services vault
az backup vault create \
  --resource-group az104-lab-rg \
  --name az104-vault \
  --location eastus

# Enable backup for a VM
az backup protection enable-for-vm \
  --resource-group az104-lab-rg \
  --vault-name az104-vault \
  --vm myVM \
  --policy-name DefaultPolicy

# Trigger an on-demand backup
az backup protection backup-now \
  --resource-group az104-lab-rg \
  --vault-name az104-vault \
  --container-name myVM \
  --item-name myVM \
  --retain-until 2026-05-28
```

> **Monitoring Gotcha: Alert Fatigue**
>
> **The Problem:** You set up 50 alert rules. Within a week, the team ignores all emails because most are noise.
>
> **The Fix:** Use severity levels (0=Critical through 4=Verbose). Only page on-call for Sev 0-1. Start with few high-confidence alerts and add more as you understand the baseline.

---

## 7. Cross-Domain Decision Tables

### 7.1 Identity and Governance

| Need | Best tool |
|------|-----------|
| Allow a user to manage resources | RBAC role at scope |
| Allow a user to assign roles | User Access Administrator or Owner |
| Block disallowed resource types | Azure Policy (deny) |
| Audit without blocking | Azure Policy (audit) |
| Prevent accidental delete | Lock (CanNotDelete) |
| Track cost owner | Tags |
| Set spending limit | Budget + action group |

### 7.2 Storage

| Need | Best tool |
|------|-----------|
| Temporary limited access | SAS token |
| Identity-based access | Entra ID + Storage RBAC data roles |
| Private network access | Private endpoint + Private DNS |
| Auto-tier old data | Lifecycle management policy |
| Protect against deletion | Soft delete |
| Legal hold / compliance | Immutability policy (WORM) |

### 7.3 Compute

| Need | Best tool |
|------|-----------|
| Full OS control | VM |
| Many identical servers | VMSS |
| Managed web app with slots | App Service |
| Quick container run | Container Instances (ACI) |
| Private image registry | Container Registry (ACR) |
| Declarative deployment | ARM template or Bicep |

### 7.4 Networking

| Need | Best tool |
|------|-----------|
| HTTP-aware routing with WAF | Application Gateway |
| TCP/UDP distribution | Load Balancer (L4) |
| Secure admin access | Azure Bastion |
| Private VNet connectivity | VNet Peering |
| Route through firewall | UDR |
| On-prem to Azure over internet | VPN Gateway |
| Dedicated private connection | ExpressRoute |
| Diagnose NSG blocking | Network Watcher → IP Flow Verify |

### 7.5 Monitoring

| Need | Best tool |
|------|-----------|
| Real-time trend | Metrics + dashboard |
| Deep investigation | Logs + KQL |
| Audit trail | Activity Log |
| Threshold notification | Metric alert + action group |
| Route telemetry | Diagnostic settings |
| VM backup | Azure Backup + Recovery Services Vault |

---

## 8. Exam Gotchas and Common Mistakes

| Trap | Correct thinking | Section |
|------|------------------|---------|
| Policy grants access | Policy enforces compliance; RBAC grants access | §2.8 |
| Contributor can assign roles | Cannot — use Owner or User Access Admin | §2.6 |
| ReadOnly lock only blocks config | It blocks data-plane writes too | §2.10 |
| Service principals for Azure-to-Azure | Use managed identities instead | §2.3 |
| SAS equals access key | SAS is scoped and temporary | §3.6 |
| Archive tier is instant | Rehydration takes up to 15 hours | §3.4 |
| Private endpoint works without DNS | Private DNS zone is required | §5.10 |
| Stopped VM = deallocated | Stopped still bills for compute | §4.3 |
| NSG order is random | Lower priority number wins | §5.3 |
| Health probe = app health | Probe only checks what it's configured for | §5.14 |
| Peering is transitive | It is not — each pair must be configured | §5.4 |
| LB can route by URL path | LB is L4 — use App Gateway for L7 | §5.7 |
| Creating workspace collects data | Diagnostic settings must be configured | §6.4 |

---

## 9. Final Revision Card

| Topic | One-line memory aid |
|------|----------------------|
| Entra ID | Who you are |
| RBAC | What you can do and where |
| Custom roles | Actions + DataActions + AssignableScopes |
| Policy | What is allowed (deny, audit, modify) |
| Locks | What must not be changed or deleted |
| Tags | How you classify and report |
| Budgets | Spending threshold with alerts |
| SAS | Temporary delegated storage access |
| Lifecycle rules | Auto-tier or expire blobs |
| Private endpoint | Private IP to PaaS + DNS zone |
| Availability set | Host/rack failure protection |
| Availability zone | Datacenter failure protection |
| VMSS | Repeatable scale-out VMs |
| App Service | Managed web hosting with slots |
| Load Balancer | Layer 4 (TCP/UDP) distribution |
| Application Gateway | Layer 7 (HTTP) + WAF |
| NSG | Traffic filter (lower # wins) |
| Bastion | Secure admin without public IP |
| VPN Gateway | On-prem to Azure encrypted tunnel |
| ExpressRoute | Dedicated private connection |
| Network Watcher | IP Flow Verify, Next Hop, Capture |
| Metrics | Fast numeric signals |
| Logs | Deep investigation via KQL |
| Activity log | Control-plane audit trail |
| Diagnostic settings | Route telemetry to destination |
| Azure Backup | Recovery Services Vault + policies |

### 9.1 60-Second Exam Recap

If you remember only one sentence per domain:

- **Identity and governance** is about **who can do what at which scope, and what guardrails prevent mistakes**.
- **Storage** is about **how data is protected, shared, accessed, and moved between tiers**.
- **Compute** is about **choosing the right hosting model and managing availability**.
- **Networking** is about **how traffic reaches the right place safely, and how you debug when it doesn't**.
- **Monitoring** is about **what happened, where, why, and who gets notified**.

---

**Version:** 2.0 | **Last Updated:** 2026-04-28 | **Source:** Microsoft Learn AZ-104 learning paths

