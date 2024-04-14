## Azure Concepts

## PowerShell and CLI
[Azure CLI Commands](https://learn.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest)

[AZ PowerShell Commands](https://learn.microsoft.com/en-us/powershell/azure/?view=azps-11.5.0)

# Manage Azure identities and governance

## Entra ID

## Manage Entra Users and Groups

## Manage Role Based Access Control
* Permissions are additive so if a user has move than one role assigned they will get the union of both roles (no permissions will be taken away, so if one role has a specific deny and another has an allow for that same permission the allow wins)

## Manage Subscriptions and Governance

### Cost Management
* Can configure cost alerts both budget based and anomaly based
* Anomly based alerts can't be created at a billing scope only subcription scope
* Budgets have expiry dates

### Resource Locks
[MS Learn: Resource Locks Documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/lock-resources?tabs=jsons)

* Resource locks only work on control plane operations not data plane operations

Two types of locks:
* **Read-only** | Means authorized users can read a resource, but they can't delete or update it. Applying this lock is similar to restricting all authorized users to the permissions that the Reader role provides.

* **CanNotDelete** | Means authorized users can read and modify a resource, but they can't delete it.

When you cancel an Azure subscription:
* A resource lock doesn't block the subscription cancellation.

#### Who can create or delete locks

To create or delete management locks, you need access to `Microsoft.Authorization/*` or `Microsoft.Authorization/locks/*` actions. Users assigned to the Owner and the User Access Administrator roles have the required access

#### Considerations before applying locks
* A read-only lock on a storage account prevents users from listing the account keys. A POST request handles the Azure Storage List Keys operation to protect access to the account keys. The account keys provide complete access to data in the storage account. When a read-only lock is configured for a storage account, users who don't have the account keys need to use Microsoft Entra credentials to access blob or queue data. A read-only lock also prevents the assignment of Azure RBAC roles that are scoped to the storage account or to a data container (blob container or queue).

* A read-only lock or cannot-delete lock on a storage account doesn't prevent its data from deletion or modification. It also doesn't protect the data in a blob, queue, table, or file.

* A read-only lock on a resource group that contains an App Service plan prevents you from scaling up or out of the plan.

* A read-only lock on a resource group prevents you from moving existing resources in or out of the resource group.

* A cannot-delete lock on a resource or resource group prevents the deletion of Azure RBAC assignments.

### Azure Policy

[MS Learn: Azure Policy Documentation](https://learn.microsoft.com/en-us/azure/governance/policy/)

* You can assign Azure policies at all scopes: Tenant Root, Management Group, Subscription, Resource Group, Resource

* You can exlclude Azure policies at all scopes except *Tenant Root*: Management Group, Subscription, Resource Group, Resource

#### Azure Policy definition structure basics

Policy Types:  Builtin, Custom, Static (Indicates a Regulatory Compliance policy definition with Microsoft Ownership)

Mode: The `mode` is configured depending on if the policy is targeting an Azure Resource Manager property or a Resource Provider property.

**Resource Manager Modes**
* all: evaluate resource groups, subscriptions, and all resource types
* indexed: only evaluate resource types that support tags and location

We recommend that you set mode to all in most cases

**Resource Manager Modes**

The following Resource Provider modes are fully supported:
* `Microsoft.Kubernetes.Data` for managing Kubernetes clusters and components such as pods, containers, and ingresses
* `Microsoft.KeyVault.Data` for managing vaults and certificates in Azure Key Vault
* `Microsoft.Network.Data` for managing Azure Virtual Network Manager custom membership policies using Azure Policy

```{
  "properties": {
    "displayName": "Allowed locations",
    "description": "This policy enables you to restrict the locations your organization can specify when deploying resources.",
    "mode": "Indexed",
    "metadata": {
      "version": "1.0.0",
      "category": "Locations"
    },
    "parameters": {
      "allowedLocations": {
        "type": "array",
        "metadata": {
          "description": "The list of locations that can be specified when deploying resources",
          "strongType": "location",
          "displayName": "Allowed locations"
        },
        "defaultValue": [
          "westus2"
        ]
      }
    },
    "policyRule": {
      "if": {
        "not": {
          "field": "location",
          "in": "[parameters('allowedLocations')]"
        }
      },
      "then": {
        "effect": "deny"
      }
    }
  }
}
```

### Move Resources

[MS Learn: Moving Resources](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/move-resources-overview)

* When moving resources the resource id will change

* You can move Azure resources to either another Azure subscription or another resource group under the same subscription.

* The move operation doesn't support moving resources to new Microsoft Entra tenant
    * Need to transfer the ownership of the subscription
    * Associate the subscription to another Entra Id Tenant

#### Checklist before moving resources
1. The source and destination subscriptions must be active.
2. The source and destination subscriptions must exist within the same Microsoft Entra tenant.
3. If you're attempting to move resources to or from a Cloud Solution Provider (CSP) partner, see Transfer Azure subscriptions between subscribers and CSPs.
4. The resources you want to move must support the move operation
5. Some services have specific limitations or requirements when moving resources. If you're moving any of the following services, check that guidance before moving.
    * [App Services move guidance](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/move-limitations/app-service-move-limitations)
    * [Networking move guidance](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/move-limitations/networking-move-limitations)
6. The destination subscription must be registered for the resource provider of the resource being moved
7. Before moving the resources, check the subscription quotas for the subscription you're moving the resources to
8. The account moving the resources must have at least the following permissions:
    * `Microsoft.Resources/subscriptions/resourceGroups/moveResources/action` on the source resource group.
    * `Microsoft.Resources/subscriptions/resourceGroups/write` on the destination resource group.
9. If you move a resource that has an Azure role assigned directly to the resource (or a child resource), the role assignment isn't moved and becomes orphaned.
10. For a move across subscriptions, the resource and its dependent resources must be located in the same resource group and they must be moved together.

# Implement and manage storage

# Deploy and manage Azure compute resources

# Implement and manage virtual networking

# Monitor and maintain Azure resources