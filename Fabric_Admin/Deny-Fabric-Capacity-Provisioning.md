Restrict Fabric Capacity Provisioning to only `fabric_admins` Group
===================================================================

Overview
--------

The goal: only members of the `fabric_admins` group can create Microsoft Fabric capacities. Everyone else is blocked, everywhere.
This is harder than it sounds because Azure has two separate access control systems that work in different ways:
*   **Azure Policy** is scope-based. It controls what kinds of resources can exist in a given scope (management group, subscription, resource group). It does not look at identity. A policy cannot say "allow group X, deny group Y."
*   **RBAC** is identity-based. It controls which users and groups have which permissions at a given scope. It does not look at resource type rules.
Neither one alone can express "only this group, only this resource type, anywhere in the tenant." The implementation below combines them:
1.  **Azure Policy (Deny)** at Root Management Group (**MG**) — blocks all Fabric capacity creation everywhere
2.  **Policy Exemption** on designated subscriptions — removes the policy block in specific subscriptions
3.  **Custom RBAC Role** assigned only to `fabric_admins` — grants the actual permission only to that group, only in those subscriptions

> **Key Limitation**: Azure Policy is scope-based, not identity-based. It cannot say "allow group X, deny group Y." The exemption + RBAC combination achieves identity-level control.

* * *

Layers and Effects
---------------------

| Layer | Who | Effect |
| --- | --- | --- |
| **Policy (Deny)** at Root MG | Everyone | Blocks `Microsoft.Fabric/capacities` creation |
| **Policy Exemption** on subscription | Everyone at that scope | Policy no longer blocks |
| **RBAC (Custom Role)** on subscription | Only `fabric_admins` | Only they have the `write` permission |
Unauthorized users are blocked **twice**: by policy (everywhere outside the exempted subscriptions) and by RBAC (they lack the role even inside the exempted subscriptions).

* * *

Implementation Steps
--------------------

### Step 1: Define Variables

Sets the names, IDs, and scopes used by the rest of the script. The important one is `$exemptedSubscriptionIds`, this is the allowlist of subscriptions where `fabric_admins` will be permitted to create capacities. Every other subscription in the tenant remains fully blocked.

    $rootMgId          = (Get-AzManagementGroup -GroupId (Get-AzContext).Tenant.Id).Id
    $policyName        = "Deny-Fabric-Capacity-Provisioning"
    $policyDisplayName = "Deny Fabric Capacity Provisioning"
    $policyDescription = "Denies creation of Microsoft.Fabric/capacities unless exempted. Only fabric_admins group may provision capacities in exempted scopes."
    $customRoleName    = "Fabric Capacity Administrator"
    $entraGroupName    = "fabric_admins"
    
    # Subscription(s) where fabric_admins is allowed to create capacities
    $exemptedSubscriptionIds = @(
      "4419754c-7d8c-4573-861a-2737ee397693"
      # Add more subscription IDs as needed:
      # "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    )
    

### Step 2: Create the Policy Definition at Root MG

Creates the policy rule itself but does not yet apply it to anything. The rule logic is simple: if the resource type being created is `Microsoft.Fabric/capacities`, deny the operation. The rule is defined at the Root Management Group so it can be assigned tenant-wide in the next step.

    $policyRule = @'
    {
      "if": {
        "field": "type",
        "equals": "Microsoft.Fabric/capacities"
      },
      "then": {
        "effect": "deny"
      }
    }
    '@
    
    New-AzPolicyDefinition `
      -Name $policyName `
      -DisplayName $policyDisplayName `
      -Description $policyDescription `
      -Policy $policyRule `
      -Mode "All" `
      -ManagementGroupName (Get-AzContext).Tenant.Id
    

### Step 3: Assign the Policy at Root MG

Takes the policy definition from Step 2 and applies it at the Root Management Group scope. Once this runs, the deny rule is active across every subscription in the tenant. At this point no one (including the future `fabric_admins`) can create a Fabric capacity anywhere. Steps 4 through 6 carve out the controlled exception.

    $definition = Get-AzPolicyDefinition -Name $policyName -ManagementGroupName (Get-AzContext).Tenant.Id
    
    New-AzPolicyAssignment `
      -Name $policyName `
      -DisplayName $policyDisplayName `
      -PolicyDefinition $definition `
      -Scope $rootMgId `
      -EnforcementMode Default
    

### Step 4: Create Policy Exemptions on Designated Subscriptions

Removes the policy block from each subscription in the allowlist. An exemption tells Azure "this assignment does not apply at this scope." The exemption applies to everyone in that scope, it does not single out `fabric_admins`. The identity restriction is added in Step 6 via RBAC.

ExemptionCategory **`"Waiver"`** is Azure's classification for "we are intentionally choosing to allow this," as opposed to **`"Mitigated"`**, which would mean "the risk is being addressed another way."

    $assignment = Get-AzPolicyAssignment -Name $policyName -Scope $rootMgId
    
    foreach ($subId in $exemptedSubscriptionIds) {
        $exemptionName = "Exempt-FabricAdmins-$subId"
    
        New-AzPolicyExemption `
          -Name $exemptionName `
          -DisplayName "Allow fabric_admins to provision Fabric capacities ($subId)" `
          -Description "Exempts subscription $subId so fabric_admins can create Fabric capacities" `
          -PolicyAssignment $assignment `
          -Scope "/subscriptions/$subId" `
          -ExemptionCategory "Waiver"
    
        Write-Host "Created policy exemption for subscription: $subId"
    }
    

### Step 5: Create a Custom RBAC Role

Defines a new RBAC role called `Fabric Capacity Administrator`. Built-in roles like `Contributor` grant far more than is needed, so a custom role is used to grant the minimum permissions required:

`Microsoft.Fabric/capacities/*` — full management of Fabric capacities (create, read, update, delete, scale, pause, resume)

`Microsoft.Resources/subscriptions/resourceGroups/read` — read access to resource groups, required to see where the capacity would be placed

`AssignableScopes` restricts where this role can even be granted. By limiting it to the exempted subscriptions, the role itself cannot be assigned outside the intended scope.

    $assignableScopes = $exemptedSubscriptionIds | ForEach-Object { "/subscriptions/$_" }
    
    $roleDefinition = @{
      Name             = $customRoleName
      Description      = "Can create, manage, and delete Microsoft Fabric capacities."
      Actions          = @(
        "Microsoft.Fabric/capacities/*",
        "Microsoft.Resources/subscriptions/resourceGroups/read"
      )
      NotActions       = @()
      AssignableScopes = $assignableScopes
    }
    
    New-AzRoleDefinition -Role $roleDefinition
    

### Step 6: Assign the Custom Role to `fabric_admins`

Grants the custom role to the `fabric_admins` Entra ID group on each exempted subscription. This is the final piece: the policy block is gone in these subscriptions (Step 4), and now `fabric_admins` is the only group with the underlying permission to create capacities there. Users outside the group still cannot create capacities, because they were never granted the role.

    $group = Get-AzADGroup -DisplayName $entraGroupName
    
    foreach ($subId in $exemptedSubscriptionIds) {
        New-AzRoleAssignment `
          -ObjectId $group.Id `
          -RoleDefinitionName $customRoleName `
          -Scope "/subscriptions/$subId"
    
        Write-Host "Assigned '$customRoleName' to fabric_admins on subscription: $subId"
    }
    

### Step 7: Verify

Reads back the policy assignment, exemptions, and role assignments to confirm the setup matches what was intended. Run this after the previous steps complete to catch any missing pieces before relying on the controls.

    # Check policy assignment
    Get-AzPolicyAssignment -Name $policyName -Scope $rootMgId | Format-List
    
    # Check exemptions and role assignments per subscription
    foreach ($subId in $exemptedSubscriptionIds) {
        Write-Host "`n--- Subscription: $subId ---"
    
        Write-Host "Policy Exemption:"
        Get-AzPolicyExemption -Scope "/subscriptions/$subId" | Format-List
    
        Write-Host "Role Assignment:"
        Get-AzRoleAssignment -ObjectId $group.Id -Scope "/subscriptions/$subId" |
          Where-Object { $_.RoleDefinitionName -eq $customRoleName } | Format-List
    }
    

* * *

Alternative: Tenant-Wide Access (All Subscriptions)
---------------------------------------------------

Use this variant if `fabric_admins` must create capacities in **any** subscription in the tenant without maintaining a per-subscription allowlist. The control model changes: RBAC becomes the actual enforcement mechanism, and the policy is downgraded to audit-only for monitoring.
First, widen the custom role's assignable scope to the Root MG and assign it there:

    # Assign the custom role at Root MG scope instead
    $roleDefinition.AssignableScopes = @($rootMgId)
    Set-AzRoleDefinition -Role $roleDefinition
    
    New-AzRoleAssignment `
      -ObjectId $group.Id `
      -RoleDefinitionName $customRoleName `
      -Scope $rootMgId
    

Then switch the policy from **Deny enforcement** to **audit-only**:

    Set-AzPolicyAssignment -Name $policyName -Scope $rootMgId -EnforcementMode DoNotEnforce
    

`DoNotEnforce` means the policy no longer blocks creation, but compliance violations still get recorded. The result: only `fabric_admins` has the RBAC permission to create capacities (so no one else can), and the policy provides a compliance log of every Fabric capacity that gets created tenant-wide. The tradeoff versus the primary approach: one enforcement layer instead of two, but no need to maintain a subscription allowlist.

* * *

Required Permissions to Implement
---------------------------------

These are the permissions the person running the script needs. Without them, individual steps will fail.
| Action | Required Role |
| --- | --- |
| Create policy definition at MG | `Resource Policy Contributor` on the Management Group |
| Assign policy at MG | `Resource Policy Contributor` on the Management Group |
| Create policy exemption | `Resource Policy Contributor` on the exempted scope |
| Create custom role definition | `Owner` or `User Access Administrator` on assignable scope |
| Assign roles | `User Access Administrator` or `Owner` on target scope |

* * *

Operational Considerations
--------------------------

Items to maintain after the controls are in place.
| Task | Detail |
| --- | --- |
| **Monitoring** | Set up Activity Log alerts for `Microsoft.Fabric/capacities/write` attempts so blocked and successful creations are both visible |
| **Group Governance** | Periodically audit `fabric_admins` membership in Entra ID — the security of this entire setup depends on who is in that group |
| **Exemption Scope** | Keep exemptions narrow — prefer specific subscriptions/RGs over broad scopes like management groups |
| **Naming** | Use consistent naming for policy artifacts so they are easy to find and identify in the portal and via CLI |
