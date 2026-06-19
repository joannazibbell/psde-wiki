
# WFCA External Agency Access Policy

Microsoft Fabric and Power BI - B2B Guest Access Guidance for Public Safety Data Exchange (PSDE)

Version 2.0 | May 08, 2026
Prepared by: Jerid Fortney, Architect II - Insight
Distribution: WFCA Steering Committee, Agency IT Leads


# Table of Contents

- 1. Purpose & Scope

- 2. Persona Definitions (PSDE Roles)

- 3. Access Model by Persona

- 4. B2B Guest User Limitations

- 5. Required Fabric Tenant Settings

- 6. GCC Cross-Cloud Considerations

- 7. Licensing Requirements

- 8. Capacity & Cost Implications

- 9. Risk Register

- Appendix A — Onboarding Checklist

- Appendix B — References


# 1. Purpose & Scope

This policy defines how external fire agencies access the Western Fire Chiefs Association (WFCA) Microsoft Fabric and Power BI tenant as part of the Public Safety Data Exchange (PSDE) initiative. It establishes persona-based access rules, documents known limitations of Microsoft Entra B2B guest access in Fabric [Ref 3], identifies the Fabric Admin tenant settings required to support external agency collaboration [Ref 9], and addresses the unique constraints of agencies operating on M365 GCC tenants.

This document is informed by licensing and architecture discussions held between Insight (Jerid Fortney, Sai, Cyrus), WFCA stakeholders (Joanna Zibbell, Michael), and agency representatives (Mauricio), and is grounded in Microsoft's official Fabric and Power BI B2B documentation.

## Applicable Audiences

- WFCA Fabric Administrator(s)

- WFCA Steering Committee

- Agency IT administrators and data stewards

- Insight consulting team (architecture and delivery)


# 2. Persona Definitions (PSDE Roles)

All external agency users are classified into one of three personas. The persona determines the identity model (WFCA-credentialed vs. B2B guest), the licensing requirement, and the scope of functionality available in the WFCA tenant.

| Persona | Description | Identity Model | License Requirement | Typical Count per Agency |
| --- | --- | --- | --- | --- |
| PSDE Data Steward | Agency's primary point of contact for data governance, semantic model management, pipeline monitoring, and workspace administration within the WFCA tenant. | WFCA Credential (full member account on WFCA tenant) | Power BI Pro license assigned in WFCA tenant | 1–2 |
| PSDE Developer | Builds and publishes Power BI reports, creates/edits dataflows, and develops Fabric items (lakehouses, notebooks, pipelines) for their agency's workspace. | WFCA Credential (full member account on WFCA tenant) | Power BI Pro license assigned in WFCA tenant | 1–3 |
| PSDE Consumer | Views published reports, dashboards, and apps. Does not create, edit, or publish content. Read-only access. | B2B Guest (Entra B2B invitation to WFCA tenant) | Fabric Free license (if F64+ capacity in place); otherwise Power BI Pro in WFCA tenant | Variable (10–500+) |


### Key Decision

PSDE Data Stewards and PSDE Developers receive full WFCA member accounts to avoid the significant authoring and tooling limitations that affect B2B guest users (see Section 4). PSDE Consumers access content as B2B guests, which is sufficient for viewing reports and dashboards when backed by F64+ capacity. [Ref 1] [Ref 3]


# 3. Access Model by Persona

## 3.1 Capability Matrix
| Capability | PSDE Data Steward | PSDE Developer | PSDE Consumer |
| --- | --- | --- | --- |
| View reports and dashboards | ✅ | ✅ | ✅ |
| View apps (Power BI Apps) | ✅ | ✅ | ✅ |
| Publish from Power BI Desktop to service | ✅ | ✅ | ❌ (B2B limitation) |
| Create and edit reports in Power BI service | ✅ | ✅ | ❌ |
| Connect Power BI Desktop to service semantic models | ✅ | ✅ | ❌ (B2B limitation) |
| Use Analyze in Excel | ✅ | ✅ | ❌ (B2B limitation) |
| Manage semantic models and dataflows | ✅ | ✅ (Contributor+) | ❌ |
| Create and manage Fabric items (Lakehouse, Warehouse, Pipeline, Notebook) | ✅ | ✅ | ❌ |
| Workspace Admin role | ✅ | ❌ | ❌ |
| Workspace Contributor/Member role | ✅ | ✅ | ❌ |
| Workspace Viewer role | ✅ | ✅ | ✅ |
| Install Personal Gateway | ✅ | ❌ | ❌ (B2B limitation) |
| Install org-wide published apps | ✅ | ✅ | ❌ (B2B limitation) |
| @mention in comments | ✅ | ✅ | ❌ (B2B limitation) |
| Set up email subscriptions | ✅ | ✅ | Depends on tenant setting |
| Fabric Activator alerts (Teams/email) | ✅ (internal only) | ✅ (internal only) | ❌ (external email blocked) |
| Use Power BI Mobile apps | ✅ | ✅ | ⚠️ Work/school account required |
| Use ArcGIS for Power BI visual | ✅ (requires Esri license) | ✅ (requires Esri license) | ✅ (public Living Atlas only) |


## 3.2 Workspace Architecture

Each agency receives a dedicated workspace (or set of workspaces) within the WFCA Fabric tenant. Workspaces are assigned to the shared F64 capacity. PSDE Data Stewards are granted the Admin role on their agency's workspace(s), the Domain Admin role on their domain and the Capacity Admin role on any agency-dedicated capacities. [Ref 4] PSDE Developers are granted the Contributor or Member role. PSDE Consumers are granted the Viewer role or receive access via Power BI Apps (all of these roles can be represented in an Access and Security diagram for their workspace(s)). [Ref 2]

Because Fabric items (Lakehouses, Warehouses, Notebooks, Pipelines) cannot be individually shared with B2B guest users—only reports, dashboards, semantic models, and apps support explicit item-level sharing to guests—workspace-level access is the primary mechanism for Fabric item collaboration. This reinforces the need for tight workspace segmentation per agency. [Ref 3] [Ref 4]


# 4. B2B Guest User Limitations

The following limitations apply to users accessing the WFCA tenant as Microsoft Entra B2B guest users (i.e., PSDE Consumers and any agency users not issued WFCA credentials). These are documented by Microsoft and confirmed in project architecture discussions. [Ref 3]

## 4.1 Authoring & Publishing Limitations

- Cannot publish directly from Power BI Desktop to the Power BI service: Guest users must use the Power BI service web interface and "Get Data > Upload" to publish .pbix files. This is a significant workflow friction for report developers. [Ref 3]

- Cannot connect Power BI Desktop to service semantic models or dataflows: Guests cannot use live/DirectQuery connections from Desktop to datasets hosted in the WFCA tenant service. [Ref 3]

- Cannot use Analyze in Excel: The "Analyze in Excel" feature, which creates a live-connected Excel workbook, is not available to guest users. [Ref 3]

- Cannot install a Power BI Gateway: Guests cannot install or manage an on-premises data gateway connected to the WFCA tenant. [Ref 3]

- Cannot install apps published to the entire organization: Org-wide app distribution does not extend to guest users; apps must be explicitly shared. [Ref 3]

## 4.2 Collaboration & UX Limitations

- Cannot be @mentioned in comments: Guest users cannot receive comment notifications via @mention in Power BI. [Ref 3]

- Workspace access list does not support ad hoc invites: Guests cannot be added to a workspace via ad hoc invite; they must be pre-invited via planned invite (Entra portal or PowerShell) and then added to the workspace. [Ref 3]

- Limited discoverability without "Browse" setting: If the "Guest users can browse and access Fabric content" tenant setting is disabled, guests can still access content they have permissions to, but only via direct links—not through the left-hand navigation pane. [Ref 3] [Ref 9]

- Tenant switching required: Guest users must use the tenant switcher or the "From external orgs" tab to navigate to the WFCA tenant from their home tenant. [Ref 3]

## 4.3 Fabric-Specific Limitations

- Workspace-first sharing model for Fabric items: In Fabric, guest user sharing is primarily done by sharing the workspace. Explicit sharing of particular items with guests is NOT supported except for reports, dashboards, semantic models, and apps. Lakehouses, Warehouses, Notebooks, Pipelines, and other Fabric items require workspace-level access. [Ref 3] [Ref 4]

- Fabric Activator cannot email external domains: Activator alerts (triggered by report metrics) can send messages to Teams channels or internal email addresses, but cannot send to external email domains. Agencies requiring event-based notifications must integrate via Logic Apps, Power Automate, or SAS token event streams in their own Azure environments.

- Power Automate not natively available in Fabric: Agencies cannot use Power Automate from within Fabric directly. Automation requiring Power Platform tools must be set up in the agency's own tenant and connected via endpoints.

## 4.4 Social Identity Limitations

Guest users authenticating with social/personal identities (e.g., Gmail, Outlook.com, Hotmail) face additional restrictions beyond those listed above: [Ref 3]

- Browser-only consumption in the Power BI service (no mobile apps).

- Cannot sign in where a work or school account is required.

- SSO is not supported; these users rely on one-time passcode or Microsoft Account authentication.

- Recommendation: All agency users should authenticate with their organizational work/school account (Entra ID) rather than personal email addresses.

## 4.5 Row-Level Security (RLS) Consideration

External guest user UPNs may display differently in Power BI than in Microsoft Entra ID (e.g., "ABC#EXT#@wfca.onmicrosoft.com" vs. "Live#ABC#EXT#@wfca.onmicrosoft.com"). If RLS rules are keyed on UPN, they must account for these variations to avoid unintended data exposure or access denial. [Ref 3]


# 5. Required Fabric Tenant Settings

The following tenant settings must be configured by the WFCA Fabric Administrator in the Admin Portal (Admin Portal → Tenant Settings → Export and Sharing Settings, and related sections) to support the external agency access model described in this policy. [Ref 9]

## 5.1 Required Settings (Must Be Enabled)

| # | Tenant Setting | Location | Purpose / Notes |
| --- | --- | --- | --- |
| 1 | Guest users can access Microsoft Fabric | Export and sharing settings | Master switch. If OFF, all B2B guests receive an error accessing any Fabric content. Must be enabled for PSDE Consumers. |
| 2 | Users can invite guest users to collaborate through item sharing and permissions | Export and sharing settings | Allows WFCA users to invite external guests via sharing/permissions/subscription experiences. Inviter must also have the Entra Guest Inviter role. |
| 3 | Guest users can browse and access Fabric content | Export and sharing settings | Enables full left-nav browsing for guests. If OFF, guests keep permissions but must use direct links. Recommended: ON for PSDE Data Stewards/Developers; evaluate for PSDE Consumers. |
| 4 | Users can create Fabric items | Microsoft Fabric settings | Ensure PSDE Data Stewards and Developers (who hold WFCA credentials) can create Fabric items in their assigned workspaces. |
| 5 | B2B guest users can set up and be subscribed to email subscriptions | Export and sharing settings | Enable if PSDE Consumers need to receive scheduled report email subscriptions. |


## 5.2 Recommended Settings (Enable Based on Operating Model)

| # | Tenant Setting | Location | Purpose / Notes |
| --- | --- | --- | --- |
| 6 | Users can see guest users in lists of suggested people | Export and sharing settings | If ON, guest users appear in people pickers. If OFF, reduces accidental oversharing but users must type full email to share. Recommendation: ON for now; revisit as agency count grows. |
| 7 | Guest users can work with shared semantic models in their own tenants | Export and sharing settings | Enables in-place semantic model sharing so guests can discover/connect/build on shared semantic models in their home tenant. Enable only if this pattern is desired. |
| 8 | Allow specific users to turn on external data sharing | Export and sharing settings | Controls who can share Power BI semantic models externally via Entra B2B. Scope to a specific security group of authorized users. |
| 9 | External data sharing (OneLake) | Export and sharing settings | Allows sharing read-only links to OneLake data with external collaborators. Enable only if WFCA plans to share OneLake data directly to agency Fabric tenants. |
| 10 | Users can accept external data shares | Export and sharing settings | Allows WFCA users to accept external data shares from agencies. Enable if bidirectional OneLake sharing is required. |
| 11 | Users can send email subscriptions to external users | Export and sharing settings | Allows subscriptions to be sent to non-B2B external email addresses. Evaluate security implications before enabling. |


## 5.3 Settings to Restrict or Disable

| # | Tenant Setting | Location | Purpose / Notes |
| --- | --- | --- | --- |
| 12 | Publish to web | Export and sharing settings | STRONGLY RECOMMENDED: Disable or restrict to specific admins only. "Publish to web" creates unauthenticated public embeds. Inappropriate for a public-sector consortium handling agency data. |
| 13 | Export to Excel / Export to .csv / Download reports (.pbix) | Export and sharing settings | Evaluate whether PSDE Consumers should be able to export data. Scope restrictions via security groups if needed. |
| 14 | Allow shareable links to grant access to everyone in your organization | Export and sharing settings | Consider disabling to prevent "everyone in org" links from being created, which could inadvertently include all guest users. |


# 6. GCC Cross-Cloud Considerations

The majority of WFCA member agencies operate on M365 GCC tenants. The WFCA tenant is a commercial (non-GCC) tenant. This creates a cross-cloud boundary that introduces specific constraints.

## 6.1 Key Constraints

- "Bring Your Own License" does not work across clouds: GCC agencies' existing Power BI Pro licenses are NOT honored in the WFCA commercial tenant. Agencies cannot transfer or reuse their GCC licenses. WFCA must assign new licenses or provide F64+ capacity coverage. [Ref 1] [Ref 3]

- Fabric is not available in GCC (as of May 2026): Microsoft Fabric (OneLake, Lakehouse, Warehouse, Spark, Pipelines, etc.) is not available in Office 365 GCC, GCC High, or DoD environments. Only Power BI (via the Fabric portal) is available in GCC. This is a key driver for the "federated" approach of hosting analytics workloads in the WFCA commercial tenant. [Ref 1]

- Cross-cloud B2B requires explicit Entra configuration: Both the WFCA commercial tenant and each GCC agency tenant must enable cross-cloud settings in Microsoft Entra ID (External Identities → Cross-tenant access settings → Microsoft cloud settings). Domain name lookup is not supported cross-cloud; Tenant ID must be used. [Ref 3]

- Cross-cloud sharing with security groups does not work: Inviting a security group from a different cloud (e.g., GCC group invited to commercial tenant) does not grant access because the service cannot resolve group members across clouds. Individual user invitations are required. [Ref 3]

- Cannot invite new external users through Power BI sharing UX cross-cloud: The in-product "share" and "subscribe" experiences cannot create new guest invitations across clouds. Guest users must be pre-invited via Entra (planned invite approach). [Ref 3]

- SSO for commercial agencies (e.g., Orange County): Agencies on commercial M365 tenants (not GCC) can use SSO with their Entra IDs and may be able to bring their own Power BI Pro licenses. Orange County is the pilot agency for this B2B model. [Ref 1] [Ref 3]

## 6.2 Entra Cross-Cloud Configuration Steps (Summary)

- In the WFCA commercial tenant: Entra ID → External Identities → Cross-tenant access settings → Microsoft cloud settings → Enable "Microsoft Azure Government."

- Add the GCC agency's Tenant ID to Organizational Settings. Configure inbound B2B collaboration access (allow specific users/groups or all).

- In each GCC agency tenant: Enable "Microsoft Azure Commercial" in cloud settings. Add the WFCA commercial Tenant ID. Configure outbound B2B collaboration access.

- Configure Conditional Access in the WFCA tenant for cross-cloud guests (require MFA; optionally trust MFA claims from the GCC tenant to reduce double-MFA prompts).

- Invite GCC users to the WFCA tenant using their Tenant ID-based UPN (email-based lookup is not supported cross-cloud).


# 7. Licensing Requirements

## 7.1 Licensing by Persona

| Persona | License Required | Assigned Where | Notes |
| --- | --- | --- | --- |
| PSDE Data Steward | Power BI Pro | WFCA tenant | Pro required for creating/editing Power BI items and for Admin/Member/Contributor workspace roles. |
| PSDE Developer | Power BI Pro | WFCA tenant | Pro required for publishing, editing, and Contributor/Member workspace roles. |
| PSDE Consumer (F64+ capacity) | Fabric Free | WFCA tenant | With F64+ capacity, Viewer-role users can consume Power BI content with only a Fabric Free license. No paid license required for viewing. |
| PSDE Consumer (below F64) | Power BI Pro | WFCA tenant | If capacity is scaled below F64, all viewers need Pro licenses. This is a critical risk if capacity is scaled down. |


## 7.2 Key Licensing Principles

- GCC agencies' existing Power BI licenses CANNOT be used in the WFCA commercial tenant due to cross-cloud isolation. WFCA must procure and assign Pro licenses for Data Stewards and Developers. [Ref 1] [Ref 3]

- F64 capacity negates the need for individual viewer (Pro) licenses for PSDE Consumers, provided the workspace is assigned to F64+ capacity and the user has the Viewer role. [Ref 1] [Ref 6]

- Any user who needs to create, publish, or edit Power BI items (reports, semantic models, dashboards, dataflows) in a shared workspace requires a Power BI Pro license, regardless of capacity size. The F64+ "free viewer" entitlement applies specifically to users with the Viewer role. A user assigned a higher workspace role (Member, Contributor, Admin) with only a Free license can technically browse the workspace on F64+ capacity, but will be prompted to upgrade when attempting to create a Power BI artifact. [Ref 1] [Ref 2] [Ref 6]

- For non-Power BI Fabric items (Lakehouses, Warehouses, Notebooks, Pipelines), a Fabric Free license is sufficient to create and edit these items in any Fabric capacity workspace, regardless of workspace role. This means PSDE Developers technically need Pro only for their Power BI authoring activities, but Pro is assigned to them regardless to ensure full functionality. [Ref 1] [Ref 4]

- Suggested allocation: 2 Power BI Pro licenses per GCC agency (1 Data Steward + 1 Developer). Additional Pro licenses as needed.

- Orange County (commercial tenant) may be able to use their existing Pro licenses once they upgrade to E5, as BYOL works within the same cloud. [Ref 1] [Ref 3]


# 8. Capacity & Cost Implications

## 8.1 F64 Capacity Rationale

The F64 SKU is the minimum capacity that provides "free viewer" entitlement (Fabric Free license users can view Power BI content in Viewer role). This is equivalent to the legacy P1 Premium capacity. Below F64, every viewer needs a paid Pro license, which is economically unfeasible for onboarding hundreds of agency consumers. [Ref 1]

## 8.2 Scaling Risks

- Entitlement lag when scaling: When F64 capacity is scaled down below F64 and licensing, entitlement changes don't propagate immediately; some lag should be expected. During this window, viewers may lose access to reports. [Ref 1] [Ref 8]

- Scaling below F64 breaks viewer entitlement: If capacity is scaled to F32 or below (even temporarily), all PSDE Consumers without Pro licenses will lose access to reports. This means that reports will not be available for Free-license viewers. [Ref 1] [Ref 6]

- PAYGO vs. Reserved pricing: Running an F32 for a month on PAYGO costs approximately the same as reserving an F64. Reservation is strongly recommended for cost efficiency. [Ref 8]

- Annual licensing commitments: Fabric capacity licensing commitments are annual. Resizing mid-cycle is not possible, so agencies should plan capacity needs carefully to avoid paying for licensing and premium Fabric capacity simultaneously. [Ref 1] [Ref 5]

## 8.3 Chargeback & Fair Share Model

Capacity is shared across all agencies on the WFCA tenant. The architecture uses dedicated subscriptions and resource groups per agency to enable individual capacity scaling and chargeback tracking. Capacity metrics are installed to monitor agency-level utilization. A fair share model with overage alerting is recommended. If an Agency's utilization justifies it, a dedicated capacity may be spun up for that specific agency. [Ref 8]

## 8.4 Cost Estimates

| Item | Estimated Monthly Cost | Notes |
| --- | --- | --- |
| F64 Capacity (Reserved) | ~$5,500/mo | With reservation discount (vs. ~$8,000 list) |
| Power BI Pro License | ~$14/user/mo | Per Data Steward / Developer |
| F64 "Tipping Point" | ~200 viewers | At ~200 viewers, F64 becomes more economical than per-user Pro licensing for consumers |


# 9. Risk Register

| Risk ID | Risk | Impact | Likelihood | Mitigation |
| --- | --- | --- | --- | --- |
| R1 | Workspace blast radius — Fabric items require workspace-level sharing to guests, increasing unintended data exposure | High | Medium | Strict workspace segmentation per agency; limit Fabric item access to WFCA-credentialed users only |
| R2 | Scaling below F64 causes viewer lockout — Free-license PSDE Consumers lose report access | High | Medium | Maintain F64 minimum during business hours; document entitlement lag in runbook; alert on scaling events |
| R3 | GCC cross-cloud license confusion — Agencies assume their GCC Pro licenses work in WFCA commercial tenant | Medium | High | Proactive communication to agency IT; include in onboarding documentation; coordinate with Joanna/Dave Van B on procurement |
| R4 | RLS bypass due to UPN format mismatch — Guest UPNs display differently, causing RLS rules to fail | High | Low | Test RLS with guest accounts during development; use USERPRINCIPALNAME() and EMAIL() DAX functions; document expected UPN formats |
| R5 | Oversharing via "Everyone in org" links — Shareable links could inadvertently include all guest users | Medium | Medium | Disable "Allow shareable links to grant access to everyone in your organization" tenant setting, or scope carefully |
| R6 | Export of sensitive data by guest users — Consumers export data via Excel/CSV/PDF | Medium | Medium | Evaluate export tenant settings; scope export permissions via security groups; apply sensitivity labels where appropriate |
| R7 | Cross-cloud security group resolution failure — GCC security groups cannot be resolved in commercial tenant | Medium | High | Use individual user invitations for GCC agencies; do not rely on cross-cloud security group sharing |
| R8 | Double-MFA fatigue for cross-cloud guests — Users prompted for MFA in both tenants | Low | High | Configure MFA trust in Entra cross-tenant access settings to accept claims from partner tenant |
| R9 | Perceived double-payment for Pro licenses — Agencies object to purchasing Pro licenses when they already have them in GCC | Medium | High | Transparent communication about cross-cloud isolation; explore bundled pricing; position F64 viewer entitlement as cost offset |
| R10 | Fabric Activator cannot alert external agencies — Alerts limited to internal email/Teams | Medium | Medium | Document limitation; provide endpoint integration guidance for agencies to build their own alerting via Logic Apps or Power Automate in their own tenant |


# Appendix A — Onboarding Checklist

Use this checklist when onboarding a new agency to the WFCA Fabric tenant.

## Pre-Onboarding (WFCA Admin / Insight)

- Confirm agency tenant type (GCC vs. Commercial) and document Tenant ID.

- If GCC: Configure cross-cloud B2B settings in both WFCA and agency Entra tenants (see §6.2).

- Determine persona assignments: identify PSDE Data Stewards, Developers, and Consumers by name/role.

- Procure Power BI Pro licenses in WFCA tenant for Data Stewards and Developers.

- Create WFCA member accounts for Data Stewards and Developers; assign Pro licenses.

- Create agency workspace(s) in WFCA Fabric tenant; assign to F64 capacity.

- Assign workspace roles: Admin (Data Stewards), Contributor/Member (Developers).

- Verify all required tenant settings are enabled.

## Consumer Onboarding (B2B Guest Invite)

- Invite PSDE Consumers as B2B guests via Entra planned invite (individual invites for GCC agencies; security groups allowed for commercial agencies).

- Provide consumers with the WFCA Tenant URL for access.

- Consumer completes two-step acceptance process (email invitation → sign-in consent).

- Assign Viewer role on appropriate workspace(s) or share via Power BI App.

- Verify consumer can access reports via tenant switcher or direct link.

- Confirm RLS is functioning correctly for guest UPN formats.

## Post-Onboarding Validation

- Data Steward can publish from Power BI Desktop to WFCA service.

- Developer can create/edit reports and Fabric items in agency workspace.

- Consumer can view reports and dashboards (Viewer role).

- Capacity metrics are tracking agency utilization.

- Conditional Access policies are enforcing MFA for cross-cloud guests.

- Document any agency-specific exceptions or deviations from this policy.


# Appendix B — References

The following Microsoft Learn documentation sources were used to inform this policy. In-text reference markers (e.g., [Ref 1]) link to the corresponding entry below.

[Ref 1]  "Understand Microsoft Fabric Licenses"    Source: Microsoft Learn  |  Last Updated: February 23, 2026    https://learn.microsoft.com/en-us/fabric/enterprise/licenses

[Ref 2]  "Roles in Workspaces in Power BI"    Source: Microsoft Learn  |  Last Updated: February 24, 2026    https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-roles-new-workspaces

[Ref 3]  "Distribute Power BI Content to External Guest Users Using Microsoft Entra B2B"    Source: Microsoft Learn  |  Last Updated: 2026    https://learn.microsoft.com/en-us/power-bi/enterprise/service-admin-azure-ad-b2b

[Ref 4]  "Roles in Workspaces in Microsoft Fabric"    Source: Microsoft Learn  |  Last Updated: April 24, 2026    https://learn.microsoft.com/en-us/fabric/fundamentals/roles-workspaces

[Ref 5]  "Power BI Licensing Guide for Organizations"    Source: Microsoft Learn  |  Last Updated: November 20, 2025    https://learn.microsoft.com/en-us/fabric/enterprise/powerbi/service-admin-power-bi-licensing

[Ref 6]  "Licenses and Subscriptions for Business Users"    Source: Microsoft Learn  |  Last Updated: January 2, 2026    https://learn.microsoft.com/en-us/power-bi/fundamentals/end-user-license

[Ref 7]  "Workspaces in Power BI"    Source: Microsoft Learn  |  Last Updated: October 2, 2025    https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-new-workspaces

[Ref 8]  "Manage Your Fabric Capacity"    Source: Microsoft Learn  |  Last Updated: March 6, 2026    https://learn.microsoft.com/en-us/fabric/admin/capacity-settings

[Ref 9]  "About Tenant Settings in Microsoft Fabric"    Source: Microsoft Learn  |  Last Updated: 2026    https://learn.microsoft.com/en-us/fabric/admin/about-tenant-settings



— End of Document —

