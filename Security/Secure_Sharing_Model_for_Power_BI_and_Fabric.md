# Overview
       
This post explains the different governance and security levers that may need to be implemented to securely share data within Microsoft Fabric environments, particularly when working with sensitive public safety, healthcare, or operational data. Although Power BI is now part of Microsoft Fabric rather than a separate environment, report-layer (semantic model) security and backend (Fabric item, SQL analytics endpoint, and OneLake) security remain distinct planes — securing a report does not secure the underlying data, which is why each layer below must be addressed independently.

---
# Align permissions to PSDE personas

|Persona| Description |
|--|--|
|Data Steward |Agency's primary point of contact for data governance, semantic model management, pipeline monitoring, and workspace administration within the WFCA tenant.|
|Developer|Builds and publishes Power BI reports, creates/edits dataflows, and develops Fabric items (lakehouses, notebooks, pipelines) for their agency's workspace.  |
|Consumer  |Views published reports, dashboards, and apps. Does not create, editor, or publich content. Read-only access.

--- 
#Restrict workspace access to Data Stewards and Developers
      
>_This is one of the most important governance controls in Fabric._

**Workspace Access Should Be Limited To**
*   PSDE Data Stewards
*   PSDE Developers
*   Platform administrators

**Consumers Should Not Receive**
*   Any workspace role higher than Viewer (Member, Contributor, or Administrator)
*   Note that Viewer is not a data-access boundary. A workspace Viewer is automatically added to each lakehouse’s default OneLake reader role and can query every table in every lakehouse in the workspace through the SQL analytics endpoint, independent of any report. Treat Viewer as a reporting role only, and keep Consumers out of any workspace that contains lakehouses or warehouses (see Sections 2 and 8).

**Preferred Consumer Access**
*   Power BI App access
*   Limited Viewer access only when operationally necessary

>_Workspace membership can unintentionally expose:_
>*   semantic models
>*  lineage
>*  backend Fabric artifacts
>*  export functionality
>*  engineering resources

**If Consumers are given access to workspaces, create separate engineering and consumption workspaces:**

**Engineering Workspace**

_Contains:_
*   Lakehouses
*   Warehouses
*   Pipelines
*   Notebooks
*   Semantic models

_Access limited to:_
*   PSDE Data Stewards
*   PSDE Developers

**Consumption Workspace**

_Contains:_
*   Reports
*   Dashboards
*   Power BI Apps
*   Curated thin reports

_Primarily intended for:_
*   PSDE Consumers

---
#Share through apps instead of direct report sharing

      
**Recommended Distribution Model**
1. Publish a Power BI App
1. Assign Microsoft Entra security groups
1. Distribute access through the App

**Avoid**
*   Direct report sharing
*   Broad organizational sharing links
*   Individual user assignments
*   Consumer workspace membership

---
#Restrict semantic model permissions and build access
      
Semantic model permissions are one of the most important security controls within Microsoft Power BI and Microsoft Fabric.

Permissions granted to reports do not automatically secure the underlying semantic model. Users with elevated semantic model permissions may still be able to inspect, export, or query data outside of the intended report experience.
      
**PSDE Consumers Should Not Have**
*   Build permission
*   Reshare permission
*   Write permission
*   Direct backend access unless operationally required

**Preferred Access Pattern**
*   Read-only access through Power BI Apps
*   App-only consumption whenever possible
*   Limited Viewer access only when necessary

>_**Important**_
>When sharing reports or semantic models, disable:
>“Allow recipients to build content with the data associated with this report”

**Risks When Build Permission Is Enabled**

Users may be able to:
*   Create new reports
*   Connect through Analyze in Excel
*   Inspect semantic model structures
*   Access underlying tables
*   Query data outside of intended report filters
*   Use external tools against the semantic model
*   Bypass report visual restrictions
This is one of the most common causes of accidental data exposure.

**Additional Guidance**

Hidden tables, hidden fields, and hidden report pages are usability features, not security controls.
Semantic model permissions should be combined with:
*   Row-Level Security (RLS)
*   Object-Level Security (OLS)
*   backend access controls
*   least privilege access practices
to help ensure users only access the data appropriate for their role and operational responsibility.

---
#Implement Row-Level Security (RLS)

Row-Level Security (RLS) is used to control which records a user is allowed to access. This helps ensure users only have access to the data appropriate for their role, agency, department, or operational responsibility.

**How It Works**
*   RLS is typically implemented within:
*   Power BI semantic models
*   SQL databases or warehouses
*   SQL views
*   Supported SQL endpoints

Reports inherit these security controls from the underlying data source or semantic model.

Direct Lake reports are an exception: they read Delta data directly from OneLake and do not pass through SQL-endpoint security. Row-level or object-level security defined at the SQL analytics endpoint forces a Direct Lake query to fall back to DirectQuery, and under a fixed-identity connection that security is evaluated as the fixed identity rather than the consumer — which can silently bypass per-user RLS. For Direct Lake, define RLS in the semantic model, or use Direct Lake on OneLake with OneLake Security so that OLS and RLS are enforced consistently across every access path.

>_**Important**_
Because users may attempt to access data through methods other than reports, security should be evaluated across the full data architecture and not solely at the report layer. Report filters and slicers alone are not security controls. RLS should be implemented at the semantic model or database layer to help prevent users from accessing records outside of their authorized scope.
---
# Use Object-Level Security(OLS) for sensitive data

RLS filters rows.
OLS hides entire tables or columns.

**Recommended Uses for OLS**
*   Patient names
*   Date of birth
*   Addresses
*   Incident narratives
*   Personally identifiable information
*   Sensitive operational identifiers

This is especially important in healthcare and public safety environments.

---

# Minimize PHI within semantic models

>**The safest sensitive data is data that never enters the semantic model.**

**Recommended Practices**
*   Aggregate upstream
*   Tokenize identifiers
*   Hash IDs
*   Remove unnecessary sensitive fields

**If Reports Only Require**
*   Counts
*   Trends
*   Aggregations
*   KPIs

Then patient-level records should not exist in the model.

---

# Independently secure backend Fabric resources

Power BI report and semantic model security does not automatically secure backend Microsoft Fabric resources. Users with backend access may be able to bypass reports entirely and directly access underlying data.

**Backend Resources That Require Separate Security Review**
*   Lakehouses
*   Warehouses
*   SQL Endpoints
*   OneLake shortcuts
*   Notebooks
*   Pipelines
*   Dataflows
*   Engineering workspaces

**Required Actions**

_Lakehouses_
Review and restrict:
*   workspace membership
*   SQL analytics endpoint permissions
*   shortcut access
*   notebook access

PSDE Consumers should generally not have direct access to Lakehouses.

Because workspace Viewer alone grants this access — every workspace role places the user in the lakehouse’s default OneLake reader role with read access to the SQL analytics endpoint — restricting it requires an explicit control: remove Consumers from the lakehouse’s default reader role under Manage OneLake data access, or apply a T-SQL DENY at the SQL analytics endpoint, or (preferred) keep Consumers in a consumption workspace that contains no lakehouses or warehouses at all.

_Warehouses and SQL Endpoints_
Restrict:
*   SQL endpoint permissions
*   schema access
*   direct query access
*   external tool connectivity

Only authorized Developers and Data Stewards should have direct SQL access.

_OneLake Shortcuts_
Review:
*   where shortcuts point
*   who can access the source data
*   whether shortcuts expose sensitive backend data into broader workspaces

Do not assume shortcut security automatically changes when shared into another workspace.

_Notebooks and Pipelines_
Restrict:
*   editing permissions
*   execution permissions
*   engineering workspace access
*   automation identities

These resources should generally be limited to PSDE Developers and Data Stewards.

**Workspaces**

Limit workspace membership to users who require backend engineering access.

PSDE Consumers should typically consume content through Power BI Apps rather than direct workspace access.

>_**Important**_
>Backend access should follow least privilege principles and be reviewed separately from report-sharing permissions.
>A user who cannot access a report may still be able to access the underlying data if backend permissions have been granted elsewhere.

---
# Use Microsoft Entra groups for access management

The PSDE platform uses Microsoft Entra security groups to manage access across domains, workspaces, and Fabric resources.

**PSDE Platform Responsibilities**

WFCA Fabric administrators are responsible for:
*   establishing the core PSDE security architecture
*   configuring and managing PSDE access groups
*   mapping agency groups to PSDE roles and permissions
*   supporting cross-tenant group synchronization
*   maintaining consistent governance practices across the platform

**Agency Responsibilities**

Agencies are responsible for:
*   identifying the appropriate users within their organization
*   managing membership within their local agency groups
*   requesting access changes when needed
*   following established PSDE persona and security guidance

---

# Test with a real low-privilege PSDE Consumer account

Most security failures occur because administrators test with elevated privileges.

**Validate**
*   Export capabilities
*   Analyze in Excel access
*   Semantic model visibility
*   SQL endpoint access
*   OneLake access
*   Lineage visibility

Assume users will attempt every available action.
