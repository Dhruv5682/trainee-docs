[🏠 Home](../README.md) · [Azure](README.md)

# ☁️ AZ-104 Azure Administrator — Practice Lab Pack

> **Audience:** AZ-104 candidates who want implementation practice  
> **Focus:** Portal, CLI, and PowerShell-style scenario work  
> **Tip:** Use a free trial or sandbox subscription. The goal is deliberate practice, not a perfect production build.

## Table of Contents

1. [Lab 0 — Prep and Lab Rules](#1-lab-0--prep-and-lab-rules)
2. [Lab 1 — Identity, Scope, and Governance](#2-lab-1--identity-scope-and-governance)
3. [Lab 2 — Storage Access and Data Protection](#3-lab-2--storage-access-and-data-protection)
4. [Lab 3 — Compute and Networking](#4-lab-3--compute-and-networking)
5. [Lab 4 — Monitoring and Alerting](#5-lab-4--monitoring-and-alerting)
6. [Practice Questions](#6-practice-questions)
7. [Answer Checklists](#7-answer-checklists)
8. [Lab Gotchas](#8-lab-gotchas)

## 1. Lab 0 — Prep and Lab Rules

Use one resource group per lab. Name things clearly and delete them when done. The goal is to learn the scope boundaries and the operational flow, not to keep the environment forever.

### Walkthrough Pattern

Each lab below uses the same flow:

1. Do it in the portal first so you can see the knobs.
2. Repeat it with CLI or PowerShell so the configuration becomes repeatable.
3. Verify the result from a different angle.

### Suggested Base Layout

| Item | Example |
|------|---------|
| Resource group | `az104-lab-rg` |
| Region | `eastus` or your nearest region |
| Naming prefix | `az104` |
| Default network | One VNet with isolated subnets |

### Allowed Tools

| Tool | Use |
|------|-----|
| Portal | Learn resource layout and settings |
| Azure CLI | Repeatable configuration |
| PowerShell | Azure administrative automation |
| Cloud Shell | Browser-based authenticated shell |

## 2. Lab 1 — Identity, Scope, and Governance

### Objective

Create a governance boundary that uses groups, RBAC, policy, tags, and a lock.

### Scenario

You need to allow a team to manage a resource group, but only within a constrained naming and location standard.

### Tasks

1. Create a resource group.
2. Create a group in Microsoft Entra ID.
3. Assign the group Contributor on the resource group scope.
4. Add a policy assignment that restricts a disallowed location or resource type.
5. Add tags for owner and environment.
6. Add a delete lock to one critical resource.

### Portal Walkthrough

1. Open the Azure portal and create a new resource group.
2. Go to Microsoft Entra ID and create a security group.
3. Open the resource group and assign the Contributor role to the group.
4. Open Policy and assign a built-in deny policy for location or resource type.
5. Add tags from the resource group blade.
6. Add a lock on a resource or the resource group.

### CLI / PowerShell Walkthrough

```bash
# Login and select subscription
az login
az account set --subscription "<subscription-id-or-name>"

# Create a resource group
az group create \
	--name az104-lab-rg \
	--location eastus

# Create a security group requires Microsoft Graph or portal handling depending on tenant policy
# For the lab, create the group in the portal, then continue with RBAC in CLI.

# Assign Contributor at resource group scope
az role assignment create \
	--assignee-object-id <group-object-id> \
	--assignee-principal-type Group \
	--role Contributor \
	--scope /subscriptions/<subscription-id>/resourceGroups/az104-lab-rg

# Tag the resource group
az group update \
	--name az104-lab-rg \
	--set tags.owner=az104 tags.environment=lab

# Add a CanNotDelete lock
az lock create \
	--name rg-delete-lock \
	--lock-type CanNotDelete \
	--resource-group az104-lab-rg
```

### Verification

| Check | What to run |
|------|-------------|
| RBAC | Open Access control (IAM) and confirm the group assignment |
| Policy | Try creating a disallowed resource and confirm denial |
| Lock | Attempt to delete the locked scope and confirm rejection |

### Expected Outcome

- The group can work inside the scope you assigned.
- The policy blocks noncompliant resources.
- The lock prevents accidental deletion.

### Success Check

| Check | What to verify |
|------|----------------|
| RBAC | The group can create and manage resources in the scoped resource group |
| Policy | A denied deployment fails for a noncompliant resource |
| Lock | Deletion is blocked until the lock is removed |

## 3. Lab 2 — Storage Access and Data Protection

### Objective

Configure a storage account with multiple access methods and recovery protections.

### Tasks

1. Create a storage account.
2. Create a blob container and a file share.
3. Generate a SAS token with limited permissions and expiry.
4. Compare SAS access with account key access.
5. Enable soft delete or versioning where applicable.
6. Configure lifecycle rules for tiering or expiration.
7. Test service endpoint or private endpoint access if available in your lab.

### Portal Walkthrough

1. Create a storage account from the portal.
2. Create a blob container and file share.
3. Open Shared access signature and generate a limited SAS token.
4. Enable soft delete or versioning in the data protection section.
5. Add a lifecycle management rule.
6. Open Networking and restrict the account with a private endpoint or firewall rule.

### CLI / PowerShell Walkthrough

```bash
# Create storage account
az storage account create \
	--name az104stor$RANDOM \
	--resource-group az104-lab-rg \
	--location eastus \
	--sku Standard_LRS \
	--kind StorageV2

# Create a blob container
az storage container create \
	--name lab-container \
	--account-name <storage-account-name> \
	--auth-mode login

# Generate a SAS token for a blob container
az storage container generate-sas \
	--name lab-container \
	--account-name <storage-account-name> \
	--permissions rwd \
	--expiry 2026-04-29T00:00Z \
	--auth-mode login

# Show account keys if needed for comparison
az storage account keys list \
	--account-name <storage-account-name> \
	--resource-group az104-lab-rg
```

### Verification

| Check | What to run |
|------|-------------|
| SAS | Open the SAS URL and confirm limited access |
| Keys | Compare with full account key access |
| Protection | Delete a blob and confirm soft delete/versioning behavior |

### Validation Questions

| Question | What the exam wants |
|----------|---------------------|
| Which access method is temporary? | SAS |
| Which access method is broader? | Account key |
| Which feature protects against accidental deletion? | Soft delete / versioning |
| Which feature reduces cost over time? | Lifecycle management |

### Stretch Goal

Try to access the storage account by name from a private endpoint-enabled network and confirm the DNS record resolves correctly.

## 4. Lab 3 — Compute and Networking

### Objective

Deploy a VM, secure it with networking controls, and decide whether the workload belongs on a VM, VMSS, or App Service.

### Tasks

1. Deploy a Linux or Windows VM.
2. Remove direct public access where possible.
3. Deploy Azure Bastion or use a secure admin path.
4. Configure an NSG with inbound and outbound rules.
5. Add a public IP only if the scenario requires it.
6. Test connectivity with ping, RDP, SSH, or HTTP depending on the VM role.
7. Add a second instance conceptually and decide whether VMSS would be a better long-term fit.

### Portal Walkthrough

1. Create a virtual network and subnet.
2. Create a VM and attach the VNet/subnet.
3. Create an NSG and add inbound allow rules only for the needed protocol.
4. Deploy Bastion if your subscription supports it.
5. Review NIC effective security rules and IP configuration.

### CLI / PowerShell Walkthrough

```bash
# Create a VNet and subnet
az network vnet create \
	--resource-group az104-lab-rg \
	--name az104-vnet \
	--address-prefix 10.10.0.0/16 \
	--subnet-name app-subnet \
	--subnet-prefix 10.10.1.0/24

# Create an NSG
az network nsg create \
	--resource-group az104-lab-rg \
	--name az104-nsg

# Add an SSH rule for Linux or RDP rule for Windows
az network nsg rule create \
	--resource-group az104-lab-rg \
	--nsg-name az104-nsg \
	--name AllowSSH \
	--priority 100 \
	--direction Inbound \
	--access Allow \
	--protocol Tcp \
	--destination-port-ranges 22

# Associate the NSG to the subnet
az network vnet subnet update \
	--resource-group az104-lab-rg \
	--vnet-name az104-vnet \
	--name app-subnet \
	--network-security-group az104-nsg
```

### Verification

| Check | What to run |
|------|-------------|
| VM | Confirm the NIC is in the expected subnet |
| NSG | Review effective rules and test the port |
| Bastion | Connect without exposing a public admin port |

### Mental Checklist

| Decision | Hint |
|---------|------|
| Need OS control | VM |
| Need horizontal scale | VMSS |
| Need managed web hosting | App Service |
| Need simple container run | Container Instances |

### Network Review

| Component | Why it matters |
|----------|----------------|
| NSG | Traffic filtering |
| Bastion | Secure admin access |
| Private endpoint | Private access to PaaS |
| DNS | Name resolution for private services |

## 5. Lab 4 — Monitoring and Alerting

### Objective

Route telemetry into Log Analytics and build an alert that notifies the right people.

### Tasks

1. Enable diagnostic settings on a resource.
2. Send logs or metrics to a Log Analytics workspace.
3. Query the data with a simple KQL query.
4. Create an alert rule on a measurable condition.
5. Attach an action group.
6. Test the alert path.

### Portal Walkthrough

1. Create or select a Log Analytics workspace.
2. Open diagnostic settings on the target resource and send data to the workspace.
3. Open Logs and run a small KQL query.
4. Create a metric alert or log alert.
5. Attach an action group with email or webhook.

### CLI / PowerShell Walkthrough

```bash
# Create a Log Analytics workspace
az monitor log-analytics workspace create \
	--resource-group az104-lab-rg \
	--workspace-name az104-law \
	--location eastus

# Create an action group
az monitor action-group create \
	--resource-group az104-lab-rg \
	--name az104-ag \
	--short-name az104ag

# Example metric alert skeleton
az monitor metrics alert create \
	--name HighCPUAlert \
	--resource-group az104-lab-rg \
	--scopes /subscriptions/<subscription-id>/resourceGroups/az104-lab-rg/providers/Microsoft.Compute/virtualMachines/<vm-name> \
	--condition "avg Percentage CPU > 80" \
	--description "Alert when CPU exceeds 80%"
```

### Verification

| Check | What to run |
|------|-------------|
| Diagnostics | Confirm logs are arriving in Log Analytics |
| Query | Run a KQL query and inspect the result |
| Alert | Trigger or simulate the alert threshold |

### Example Query Thinking

| Need | Think |
|------|-------|
| Show recent errors | Filter and summarize logs |
| Track CPU trend | Metrics |
| Audit a create/delete operation | Activity log |

### Validation Questions

| Question | Answer |
|----------|--------|
| Which telemetry is best for trends? | Metrics |
| Which telemetry is best for investigations? | Logs |
| Which log tracks resource operations? | Activity log |
| Which component sends notifications? | Action group |

## 6. Practice Questions

1. A team needs access to manage resources only within one resource group. Which scope should you use?
2. A storage access requirement says the link should expire after two hours. Which access method fits best?
3. An app must be reachable privately from inside a VNet. Which connectivity feature should you choose?
4. A workload needs many identical VMs and automatic scale-out. What should you use?
5. You need to investigate why a deployment failed. Which telemetry source is the first place to check?

## 7. Answer Checklists

| Question | Answer pattern |
|----------|----------------|
| Scope for team management | Resource group scope or subscription scope, depending on blast radius |
| Temporary storage access | SAS |
| Private PaaS access | Private endpoint + DNS |
| Repeatable compute scale-out | VMSS |
| Deployment failure investigation | Activity log, then logs |

## 8. Lab Gotchas

| Problem | Likely cause |
|--------|--------------|
| Resource deploys but cannot be accessed | RBAC or NSG issue |
| Storage reachable by IP but not by name | DNS issue |
| Alert never fires | Wrong metric, threshold, or scope |
| Policy blocks deployment unexpectedly | Policy assignment scope or rule conflict |
| VM is stopped but still billed | Deallocation state not reached |

