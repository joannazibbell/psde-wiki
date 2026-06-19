# Microsoft Fabric Tenant Settings Review

**Prepared for:** Client Tenant Settings Review Meeting  
**Date:** April 14, 2026  
**Purpose:** Review, educate, and configure all Fabric Tenant Settings in the new tenant  

---

## How to Use This Document

Each setting is listed in the same order as it appears in the Admin Portal. The columns are:

| Column | Description |
|--------|-------------|
| **Setting** | The name of the tenant setting |
| **Current Value** | The current configuration in your tenant |
| **Priority** | 🔴 Critical · 🟠 High · 🟡 Medium · 🟢 Low |
| **Description** | What the setting does and why it matters |
| **Learn More** | Link to official Microsoft documentation |

**Priority Legend:**
- 🔴 **Critical** — Security, compliance, data protection, or network access. Must be reviewed and intentionally configured.
- 🟠 **High** — Core functionality, governance, external sharing, or guest access. Important for day-to-day operations.
- 🟡 **Medium** — Feature enablement, integrations, or collaboration tools. Useful but lower risk.
- 🟢 **Low** — UI preferences, preview features, or informational settings. Can be deferred.

> **Note:** Tenant setting changes can take up to 15 minutes to take effect for all users in the organization.

---

## Quick Reference: All Settings by Priority

| Priority | Setting | Category |
|----------|---------|----------|
| 🔴 Critical | [Users can create Fabric items](#1-microsoft-fabric) | Microsoft Fabric |
| 🔴 Critical | [Create workspaces](#4-workspace-settings) | Workspace Settings |
| 🔴 Critical | [Allow users to apply sensitivity labels for content](#5-information-protection) | Information Protection |
| 🔴 Critical | [Apply sensitivity labels from data sources to their data in Power BI](#5-information-protection) | Information Protection |
| 🔴 Critical | [Restrict content with protected labels from being shared via link with everyone](#5-information-protection) | Information Protection |
| 🔴 Critical | [External data sharing](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🔴 Critical | [Users can accept external data shares](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🔴 Critical | [Guest users can access Microsoft Fabric](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🔴 Critical | [Users can invite guest users to collaborate through item sharing](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🔴 Critical | [Publish to web](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🔴 Critical | [Allow specific users to turn on external data sharing](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🔴 Critical | [Web content on dashboard tiles](#13-dashboard-settings) | Dashboard Settings |
| 🔴 Critical | [Service principals can access admin APIs used for updates](#15-admin-api-settings) | Admin API Settings |
| 🔴 Critical | [Tenant-level Private Link](#21-advanced-networking) | Advanced Networking |
| 🔴 Critical | [Block Public Internet Access](#21-advanced-networking) | Advanced Networking |
| 🔴 Critical | [Apply customer-managed keys](#22-encryption) | Encryption |
| 🔴 Critical | [Users can access data stored in OneLake with apps external to Fabric](#30-onelake-settings) | OneLake Settings |
| 🔴 Critical | [Users can use Copilot and other features powered by Azure OpenAI](#32-copilot-and-azure-openai-service) | Copilot and Azure OpenAI Service |
| 🔴 Critical | [Data sent to Azure OpenAI can be processed outside your capacity's geographic region](#32-copilot-and-azure-openai-service) | Copilot and Azure OpenAI Service |
| 🔴 Critical | [Data sent to Azure OpenAI can be stored outside your capacity's geographic region](#32-copilot-and-azure-openai-service) | Copilot and Azure OpenAI Service |
| 🟠 High | [Receive email and Teams notifications for service outages or incidents](#2-help-and-support-settings) | Help and Support Settings |
| 🟠 High | [Users can try Microsoft Fabric paid features](#2-help-and-support-settings) | Help and Support Settings |
| 🟠 High | [Use semantic models across workspaces](#4-workspace-settings) | Workspace Settings |
| 🟠 High | [Define workspace retention period](#4-workspace-settings) | Workspace Settings |
| 🟠 High | [Fabric item recovery](#4-workspace-settings) | Workspace Settings |
| 🟠 High | [Automatically apply sensitivity labels to downstream content](#5-information-protection) | Information Protection |
| 🟠 High | [Allow Microsoft Purview to secure AI interactions](#5-information-protection) | Information Protection |
| 🟠 High | [Guest users can browse and access Fabric content](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟠 High | [Certification](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟠 High | [Endorse master data](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟠 High | [B2B guest users can set up and be subscribed to email subscriptions](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟠 High | [Users can send email subscriptions to external users](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟠 High | [Allow shareable links to grant access to everyone in your organization](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟠 High | [Guest users can work with shared semantic models in their own tenants](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟠 High | [Publish apps to the entire organization](#8-app-settings) | App Settings |
| 🟠 High | [Allow XMLA endpoints and Analyze in Excel with on-premises semantic models](#9-integration-settings) | Integration Settings |
| 🟠 High | [Data sent to Azure Maps can be processed outside your tenant's geographic region](#9-integration-settings) | Integration Settings |
| 🟠 High | [Data sent to Azure Maps can be processed by Microsoft Online Services Subprocessors](#9-integration-settings) | Integration Settings |
| 🟠 High | [Enable granular access control for all data connections](#9-integration-settings) | Integration Settings |
| 🟠 High | [Add and use certified visuals only (block uncertified)](#10-power-bi-visuals) | Power BI Visuals |
| 🟠 High | [Allow downloads from custom visuals](#10-power-bi-visuals) | Power BI Visuals |
| 🟠 High | [Per-user data in usage metrics for content creators](#12-audit-and-usage-settings) | Audit and Usage Settings |
| 🟠 High | [Embed content in apps](#14-developer-settings) | Developer Settings |
| 🟠 High | [Service principals can create workspaces, connections, and deployment pipelines](#14-developer-settings) | Developer Settings |
| 🟠 High | [Service principals can call Fabric public APIs](#14-developer-settings) | Developer Settings |
| 🟠 High | [Block ResourceKey Authentication](#14-developer-settings) | Developer Settings |
| 🟠 High | [Service principals can access read-only admin APIs](#15-admin-api-settings) | Admin API Settings |
| 🟠 High | [Enhance admin APIs responses with DAX and mashup expressions](#15-admin-api-settings) | Admin API Settings |
| 🟠 High | [Install template apps not listed in AppSource](#17-template-app-settings) | Template App Settings |
| 🟠 High | [Block republish and disable package refresh](#20-semantic-model-security) | Semantic Model Security |
| 🟠 High | [Configure workspace-level inbound network rules](#21-advanced-networking) | Advanced Networking |
| 🟠 High | [Configure workspace-level outbound network rules](#21-advanced-networking) | Advanced Networking |
| 🟠 High | [Configure workspace-level IP firewall rules and trusted resource instances (preview)](#21-advanced-networking) | Advanced Networking |
| 🟠 High | [Use short-lived user-delegated SAS tokens](#30-onelake-settings) | OneLake Settings |
| 🟠 High | [Authenticate with OneLake user-delegated SAS tokens](#30-onelake-settings) | OneLake Settings |
| 🟠 High | [Users can synchronize workspace items with their Git repositories](#31-git-integration) | Git Integration |
| 🟠 High | [Users can export items to Git repositories in other geographical locations](#31-git-integration) | Git Integration |
| 🟠 High | [Users can export workspace items with applied sensitivity labels to Git repositories](#31-git-integration) | Git Integration |
| 🟠 High | [Share Fabric data with your Microsoft 365 services](#25-share-data-with-your-microsoft-365-services) | Share Data with M365 Services |
| 🟠 High | [Data sent to Azure Maps can be processed outside your capacity's geographic region](#33-azure-maps-services) | Azure Maps Services |
| 🟠 High | [Users can see and work with additional workloads not validated by Microsoft](#34-additional-workloads) | Additional Workloads |
| 🟡 Medium | [ML models can serve real-time predictions from API endpoints (preview)](#1-microsoft-fabric) | Microsoft Fabric |
| 🟡 Medium | [Users can create dbt job items (preview)](#1-microsoft-fabric) | Microsoft Fabric |
| 🟡 Medium | [Enable Operations Agents (Preview)](#1-microsoft-fabric) | Microsoft Fabric |
| 🟡 Medium | [All Power BI users can see "Set alert" button](#1-microsoft-fabric) | Microsoft Fabric |
| 🟡 Medium | [Publish "Get Help" information](#2-help-and-support-settings) | Help and Support Settings |
| 🟡 Medium | [Show a custom message before publishing reports](#2-help-and-support-settings) | Help and Support Settings |
| 🟡 Medium | [Allow tenant and domain admins to override workspace assignments (preview)](#3-domain-management-settings) | Domain Management Settings |
| 🟡 Medium | [Block users from reassigning personal workspaces (My Workspace)](#4-workspace-settings) | Workspace Settings |
| 🟡 Medium | [Automatically convert and store reports using PBIR format (preview)](#4-workspace-settings) | Workspace Settings |
| 🟡 Medium | [Allow workspace admins to override automatically applied sensitivity labels](#5-information-protection) | Information Protection |
| 🟡 Medium | [Domain admins can set default sensitivity labels for their domains (preview)](#5-information-protection) | Information Protection |
| 🟡 Medium | [Users can see guest users in lists of suggested people](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Export to Excel](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Export to .csv](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Download reports](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Users can work with semantic models in Excel using a live connection](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Export reports as PowerPoint presentations or PDF documents](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Export reports as MHTML documents](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Export reports as Word documents](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Export reports as XML documents](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Users can set up email subscriptions](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Allow connections to featured tables](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Enable Microsoft Teams integration](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Enable Power BI add-in for PowerPoint](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Allow DirectQuery connections to Power BI semantic models](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Users with read or write permission can download data from notebooks](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟡 Medium | [Make promoted content discoverable](#7-discovery-settings) | Discovery Settings |
| 🟡 Medium | [Make certified content discoverable](#7-discovery-settings) | Discovery Settings |
| 🟡 Medium | [Discover content](#7-discovery-settings) | Discovery Settings |
| 🟡 Medium | [Create template organizational apps](#8-app-settings) | App Settings |
| 🟡 Medium | [Push apps to end users](#8-app-settings) | App Settings |
| 🟡 Medium | [Semantic Model Execute Queries REST API](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Users can use the Power BI MCP server endpoint (preview)](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Integration with SharePoint and Microsoft Lists](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Dremio SSO](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Snowflake SSO](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Redshift SSO](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Google BigQuery SSO](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Microsoft Entra single sign-on for data gateway](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Users can view Power BI files saved in OneDrive and SharePoint (preview)](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Users can share links to Power BI files in OneDrive and SharePoint through Desktop (preview)](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Semantic models can export data to OneLake](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Semantic model owners can choose to automatically update from OneDrive or SharePoint](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Allow non-Entra ID auth in Eventstream](#9-integration-settings) | Integration Settings |
| 🟡 Medium | [Allow visuals created using the Power BI SDK](#10-power-bi-visuals) | Power BI Visuals |
| 🟡 Medium | [AppSource Custom Visuals SSO](#10-power-bi-visuals) | Power BI Visuals |
| 🟡 Medium | [Allow access to the browser's local storage](#10-power-bi-visuals) | Power BI Visuals |
| 🟡 Medium | [Interact with and share R and Python visuals](#11-r-and-python-visuals-settings) | R and Python Visuals Settings |
| 🟡 Medium | [Usage metrics for content creators](#12-audit-and-usage-settings) | Audit and Usage Settings |
| 🟡 Medium | [Show user data in the Fabric Capacity Metrics app and reports](#12-audit-and-usage-settings) | Audit and Usage Settings |
| 🟡 Medium | [Azure Log Analytics connections for workspace administrators](#12-audit-and-usage-settings) | Audit and Usage Settings |
| 🟡 Medium | [Workspace admins can turn on monitoring for their workspaces (preview)](#12-audit-and-usage-settings) | Audit and Usage Settings |
| 🟡 Medium | [Microsoft can store query text to aid in support investigations](#12-audit-and-usage-settings) | Audit and Usage Settings |
| 🟡 Medium | [Allow service principals to create and use profiles](#14-developer-settings) | Developer Settings |
| 🟡 Medium | [Define maximum number of Fabric identities in a tenant](#14-developer-settings) | Developer Settings |
| 🟡 Medium | [Enhance admin APIs responses with detailed metadata](#15-admin-api-settings) | Admin API Settings |
| 🟡 Medium | [Create and use Gen1 dataflows](#16-gen1-dataflow-settings) | Gen1 Dataflow Settings |
| 🟡 Medium | [Publish template apps](#17-template-app-settings) | Template App Settings |
| 🟡 Medium | [Install template apps](#17-template-app-settings) | Template App Settings |
| 🟡 Medium | [Create Datamarts (preview)](#27-datamart-settings) | Datamart Settings |
| 🟡 Medium | [Users can edit semantic models in the Power BI service](#28-semantic-model-settings) | Semantic Model Settings |
| 🟡 Medium | [Scale out queries for large semantic models](#29-scale-out-settings) | Scale-out Settings |
| 🟡 Medium | [Users can sync data in OneLake with the OneLake File Explorer app](#30-onelake-settings) | OneLake Settings |
| 🟡 Medium | [Include end-user identifiers in OneLake diagnostic logs](#30-onelake-settings) | OneLake Settings |
| 🟡 Medium | [Users can sync workspace items with GitHub repositories](#31-git-integration) | Git Integration |
| 🟡 Medium | [Users can access a standalone, cross-item Power BI Copilot experience (preview)](#32-copilot-and-azure-openai-service) | Copilot and Azure OpenAI Service |
| 🟡 Medium | [Capacities can be designated as Fabric Copilot capacities](#32-copilot-and-azure-openai-service) | Copilot and Azure OpenAI Service |
| 🟡 Medium | [Only show approved items in the standalone Copilot experience (preview)](#32-copilot-and-azure-openai-service) | Copilot and Azure OpenAI Service |
| 🟡 Medium | [Workspace admins can add and remove additional workloads (preview)](#34-additional-workloads) | Additional Workloads |
| 🟡 Medium | [Capacity admins and contributors can add and remove additional workloads](#34-additional-workloads) | Additional Workloads |
| 🟢 Low | [Users can create Ontology (preview) items](#1-microsoft-fabric) | Microsoft Fabric |
| 🟢 Low | [User can create Graph (preview)](#1-microsoft-fabric) | Microsoft Fabric |
| 🟢 Low | [Users can create Digital Twin Builder (preview) items](#1-microsoft-fabric) | Microsoft Fabric |
| 🟢 Low | [Users can discover and create org apps (preview)](#1-microsoft-fabric) | Microsoft Fabric |
| 🟢 Low | [Product Feedback](#1-microsoft-fabric) | Microsoft Fabric |
| 🟢 Low | [Users can be informed of upcoming conferences](#1-microsoft-fabric) | Microsoft Fabric |
| 🟢 Low | [Detect anomalies in Real-Time Intelligence (Preview)](#1-microsoft-fabric) | Microsoft Fabric |
| 🟢 Low | [Users can create Plan (preview) items](#1-microsoft-fabric) | Microsoft Fabric |
| 🟢 Low | [Copy and paste visuals](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟢 Low | [Export reports as image files](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟢 Low | [Print dashboards and reports](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟢 Low | [Featured content](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟢 Low | [Install Power BI app for Microsoft Teams automatically](#6-export-and-sharing-settings) | Export and Sharing Settings |
| 🟢 Low | [Use ArcGIS Maps for Power BI](#9-integration-settings) | Integration Settings |
| 🟢 Low | [Use global search for Power BI](#9-integration-settings) | Integration Settings |
| 🟢 Low | [Users can use the Azure Maps visual](#9-integration-settings) | Integration Settings |
| 🟢 Low | [Map and filled map visuals](#9-integration-settings) | Integration Settings |
| 🟢 Low | [ArcGIS GeoAnalytics for Fabric Runtime](#9-integration-settings) | Integration Settings |
| 🟢 Low | [Review questions](#18-qa-settings) | Q&A Settings |
| 🟢 Low | [Synonym sharing](#18-qa-settings) | Q&A Settings |
| 🟢 Low | [Users with view permission can launch Explore](#19-explore-settings-preview) | Explore Settings (Preview) |
| 🟢 Low | [Create and use Scorecards](#23-scorecards-settings) | Scorecards Settings |
| 🟢 Low | [Help Power BI optimize your experience](#24-user-experience-experiments) | User Experience Experiments |
| 🟢 Low | [Receive notifications for top insights (preview)](#26-insights-settings) | Insights Settings |
| 🟢 Low | [Show entry points for insights (preview)](#26-insights-settings) | Insights Settings |
| 🟢 Low | [Users can use Azure Maps services](#33-azure-maps-services) | Azure Maps Services |
| 🟢 Low | [Users can use Azure Maps Weather Services (Preview)](#33-azure-maps-services) | Azure Maps Services |
| 🟢 Low | [Workspace admins can develop partner workloads](#34-additional-workloads) | Additional Workloads |

---

## 1. Microsoft Fabric

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-microsoft-fabric-tenant-settings)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Users can create Fabric items | ✅ Enabled for a subset | 🔴 Critical | Controls whether users can create production-ready Fabric items (lakehouses, warehouses, pipelines, etc.). Does not affect Power BI item creation. Can also be managed at the capacity level. | [Docs](https://learn.microsoft.com/fabric/admin/fabric-switch) |
| Users can create Ontology (preview) items | ❌ Disabled | 🟢 Low | Allows creation of ontologies to unify enterprise semantics across data, models, and logic for decision intelligence with AI agents. Preview feature. | [Docs](https://aka.ms/ontologyitem-overview) |
| User can create Graph (preview) | ❌ Disabled | 🟢 Low | Allows users to visualize data with Graph for deeper insights and richer context. Preview feature. | [Docs](https://go.microsoft.com/fwlink/?linkid=2282471) |
| Users can create Digital Twin Builder (preview) items | ❌ Disabled | 🟢 Low | Allows creation of digital twin builder items for comprehensive digital twins of real-world environments and processes. Preview feature. | [Docs](https://learn.microsoft.com/fabric/real-time-intelligence/digital-twin-builder/overview) |
| Users can discover and create org apps (preview) | ❌ Disabled | 🟢 Low | Lets users create org apps as items. If turned off, any existing org app items are hidden. The prior version of workspace apps remains available. Preview feature. | [Docs](https://learn.microsoft.com/en-us/power-bi/consumer/org-app-items/org-app-items) |
| Product Feedback | ✅ Enabled | 🟢 Low | Allows Microsoft to prompt users for feedback through in-product surveys. Participation is voluntary. Consider disabling if your organization prefers minimal Microsoft telemetry. | [Docs](https://learn.microsoft.com/fabric/fundamentals/feedback) |
| Users can be informed of upcoming conferences | ✅ Enabled | 🟢 Low | Signed-in users receive in-product notifications about upcoming conferences featuring Microsoft Fabric. | [Docs](https://go.microsoft.com/fwlink/?linkid=2306100) |
| ML models can serve real-time predictions from API endpoints (preview) | ✅ Enabled | 🟡 Medium | Enables users to create real-time predictions from model and version endpoints. Batch predictions still work when disabled. | [Docs](https://aka.ms/MLModelEndpointsLearnMore) |
| Detect anomalies in Real-Time Intelligence (Preview) | ❌ Disabled | 🟢 Low | Allows statistical detection algorithms to detect anomalies in real-time data. Preview feature. | [Docs](https://aka.ms/AD_docs) |
| Users can create dbt job items (preview) | ✅ Enabled | 🟡 Medium | Allows users to import, author, and execute dbt projects directly within Fabric for SQL-based transformation workflows without CLI setup. | [Docs](https://aka.ms/dbtjob_docs) |
| Enable Operations Agents (Preview) | ❌ Disabled | 🟡 Medium | Enables operations agents that use Azure OpenAI to create operations plans and recommend actions in response to real-time data. Data processed through Azure AI Bot Service in EU Data Boundary. | [Docs](https://go.microsoft.com/fwlink/?linkid=2338555) |
| All Power BI users can see "Set alert" button | ✅ Enabled | 🟡 Medium | Shows the "Set alert" button in reports for Fabric Activator alerts. Only users with Fabric item creation permission can actually create alerts. | [Docs](https://go.microsoft.com/fwlink/?linkid=2331953) |
| Users can create Plan (preview) items | ❌ Disabled | 🟢 Low | Allows creation of Plans in Fabric for no-code integrated planning using Power BI Semantic models, Fabric SQL and OneLake. Preview feature. | [Docs](https://aka.ms/planningdocs) |

---

## 2. Help and Support Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-help-support)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Publish "Get Help" information | ❌ Disabled | 🟡 Medium | Allows you to customize internal help and support resources shown in the Power BI help menu. Useful for directing users to your organization's own support channels. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-help-support#publish-get-help-information) |
| Receive email and Teams notifications for service outages or incidents | ❌ Disabled | 🟠 High | Mail-enabled security groups receive email and Teams notifications when the tenant is impacted by a service outage or incident. **Recommended to enable** for your operations/admin team. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-help-support) |
| Users can try Microsoft Fabric paid features | ✅ Enabled | 🟠 High | Allows users to sign up for a free 60-day Fabric trial. Consider whether you want users self-provisioning trial capacity or if you prefer controlled rollout. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-help-support#users-can-try-microsoft-fabric-paid-features) |
| Show a custom message before publishing reports | ❌ Disabled | 🟡 Medium | Displays a custom message to users before they publish a report. Useful for governance reminders (e.g., "Ensure data is not sensitive before publishing"). | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-help-support#show-a-custom-message-before-publishing-reports) |

---

## 3. Domain Management Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-domain-management-settings)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Allow tenant and domain admins to override workspace assignments (preview) | ✅ Enabled | 🟡 Medium | Allows tenant and domain admins to reassign workspaces that were previously assigned to one domain to another domain. Useful for organizational restructuring. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-domain-management-settings#allow-tenant-and-domain-admins-to-override-workspace-assignments-preview) |

---

## 4. Workspace Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/portal-workspace)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Create workspaces | ✅ Enabled for a subset | 🔴 Critical | Controls who can create app workspaces. In a new tenant, restricting this to specific security groups prevents workspace sprawl and enforces governance. Even when disabled, workspaces are created when template apps are installed. | [Docs](https://learn.microsoft.com/fabric/admin/portal-workspace#create-workspaces) |
| Use semantic models across workspaces | ✅ Enabled | 🟠 High | Allows users to use semantic models across workspaces when they have the required Build permission. Enables shared dataset patterns and reduces data duplication. | [Docs](https://learn.microsoft.com/fabric/admin/portal-workspace#use-semantic-models-across-workspaces) |
| Block users from reassigning personal workspaces (My Workspace) | ❌ Disabled | 🟡 Medium | Prevents users from reassigning their personal workspace (My Workspace) from Premium capacities to shared capacities. Consider enabling if you need to control capacity usage. | [Docs](https://learn.microsoft.com/fabric/admin/portal-workspace#block-users-from-reassigning-personal-workspaces-my-workspace) |
| Define workspace retention period | ✅ Enabled | 🟠 High | Sets a retention period for deleted workspaces before permanent deletion. Default minimum is 7 days; can be extended up to 90 days. My Workspace workspaces are retained for 30 days automatically. **Recommended to configure** to allow recovery from accidental deletions. | [Docs](https://learn.microsoft.com/fabric/admin/workspace-retention#set-up-the-retention-period-for-deleted-collaborative-workspaces) |
| Automatically convert and store reports using PBIR format (preview) | ✅ Enabled | 🟡 Medium | Automatically converts reports to PBIR (Power BI Report) format after editing. PBIR provides source control-friendly file structures, enhancing co-development and Git integration workflows. | [Docs](https://go.microsoft.com/fwlink/?linkid=2263123) |
| Fabric item recovery | ❌ Disabled | 🟠 High | When enabled, deleted Fabric items are retained for a configurable period (7–90 days) before permanent deletion. **Strongly recommended to enable** to protect against accidental deletions. Some Fabric items may not support recovery. | [Docs](https://aka.ms/FabricItemRecovery) |

---

## 5. Information Protection

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-information-protection)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Allow users to apply sensitivity labels for content | ❌ Disabled | 🔴 Critical | Enables Microsoft Purview Information Protection sensitivity labels in Fabric. Requires prerequisite steps in Purview. Labels are enforced in the tenant where applied, in PBIX files, and in supported export formats (Excel, PowerPoint, PDF). | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-information-protection#allow-users-to-apply-sensitivity-labels-for-content) |
| Apply sensitivity labels from data sources to their data in Power BI | ❌ Disabled | 🔴 Critical | Automatically inherits sensitivity labels from supported data sources into Power BI. Only applies to supported data sources. | [Docs](https://learn.microsoft.com/en-us/power-bi/enterprise/service-security-sensitivity-label-inheritance-from-data-sources) |
| Automatically apply sensitivity labels to downstream content | ❌ Disabled | 🟠 High | When a sensitivity label is changed or applied, the label is also applied to eligible downstream content automatically. Helps maintain consistent data classification. | [Docs](https://learn.microsoft.com/en-us/power-bi/enterprise/service-security-sensitivity-label-downstream-inheritance) |
| Allow workspace admins to override automatically applied sensitivity labels | ❌ Disabled | 🟡 Medium | Lets workspace admins change or remove sensitivity labels that were automatically applied (e.g., from label inheritance). | [Docs](https://learn.microsoft.com/en-us/power-bi/enterprise/service-security-sensitivity-label-change-enforcement#relaxations-to-accommodate-automatic-labeling-scenarios) |
| Restrict content with protected labels from being shared via link with everyone | ❌ Disabled | 🔴 Critical | Prevents content with protection settings in the sensitivity label from being shared via "everyone in your organization" links. Important for data loss prevention. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-information-protection#restrict-content-with-protected-labels-from-being-shared-via-link-with-everyone-in-your-organization) |
| Domain admins can set default sensitivity labels for their domains (preview) | ❌ Disabled | 🟡 Medium | Allows domain admins to set a default sensitivity label for their domains, overriding tenant-level defaults from Purview if the domain label has higher priority. | [Docs](https://learn.microsoft.com/fabric/governance/domain-default-sensitivity-label) |
| Allow Microsoft Purview to secure AI interactions | ✅ Enabled | 🟠 High | Allows Purview to access, process, and store prompts and responses (including metadata) for data security and compliance (SIT classification, DSPM for AI, Audit, Insider Risk, Communication Compliance, eDiscovery). This is a paid Purview capability, not included in Copilot pricing. | [Docs](https://go.microsoft.com/fwlink/?linkid=2296824) |

---

## 6. Export and Sharing Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-export-sharing)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| External data sharing | ❌ Disabled | 🔴 Critical | Allows users to share read-only links to OneLake data with collaborators outside your organization. Recipients can view, build on, and share data using their own licenses. **High security impact.** | [Docs](https://learn.microsoft.com/fabric/governance/external-data-sharing-overview) |
| Users can accept external data shares | ❌ Disabled | 🔴 Critical | Allows users to accept read-only data links from another organization's Fabric tenant. **Review security implications before enabling.** | [Docs](https://learn.microsoft.com/fabric/governance/external-data-sharing-overview) |
| Guest users can access Microsoft Fabric | ✅ Enabled | 🔴 Critical | Allows Entra ID B2B guest users to access Fabric and any items they have permissions to. Controls external collaboration boundary. | [Docs](https://learn.microsoft.com/fabric/enterprise/powerbi/service-admin-entra-b2b) |
| Users can invite guest users to collaborate through item sharing | ✅ Enabled | 🔴 Critical | Allows users to share Fabric items with external users and grant them permissions. After accepting, they're added as guest users in your Entra ID. | [Docs](https://learn.microsoft.com/fabric/enterprise/powerbi/service-admin-entra-b2b#invite-guest-users) |
| Guest users can browse and access Fabric content | ❌ Disabled | 🟠 High | Allows guest users to browse and request access to Fabric content. When disabled, guests can only access items explicitly shared with them. | [Docs](https://learn.microsoft.com/fabric/enterprise/powerbi/service-admin-entra-b2b) |
| Users can see guest users in lists of suggested people | ✅ Enabled | 🟡 Medium | Shows guest users in suggested people lists. When off, users must type the guest's full email to share. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-export-sharing#users-can-see-guest-users-in-lists-of-suggested-people) |
| Publish to web | ✅ Enabled | 🔴 Critical | Allows publishing public reports on the web with **no authentication required**. ⚠️ High risk: anyone with the link can view the report. **Review embed codes regularly** to ensure no confidential information is exposed. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-export-sharing#publish-to-web) |
| Copy and paste visuals | ✅ Enabled | 🟢 Low | Users can copy visuals from tiles or report visuals and paste as static images into external applications. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-export-sharing) |
| Export to Excel | ✅ Enabled | 🟡 Medium | Users can export data from visualizations or paginated reports to Excel. Consider restricting if sensitive data must stay in the platform. | [Docs](https://learn.microsoft.com/en-us/power-bi/visuals/power-bi-visualization-export-data) |
| Export to .csv | ✅ Enabled | 🟡 Medium | Users can export data from tiles, visualizations, or paginated reports to CSV files. | [Docs](https://learn.microsoft.com/en-us/power-bi/paginated-reports/report-builder/export-csv-file-report-builder) |
| Download reports | ✅ Enabled | 🟡 Medium | Users can download .pbix files and paginated reports. Consider whether you want report definitions leaving the service. | [Docs](https://learn.microsoft.com/en-us/power-bi/create-reports/service-export-to-pbix) |
| Users can work with semantic models in Excel using a live connection | ✅ Enabled | 🟡 Medium | Enables exporting data to Excel from report visuals or semantic models with a live XMLA connection (Analyze in Excel). | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-analyze-in-excel) |
| Export reports as PowerPoint presentations or PDF documents | ✅ Enabled | 🟡 Medium | Users can export reports as PowerPoint or PDF files. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-export-sharing#export-reports-as-powerpoint-presentations-or-pdf-documents) |
| Export reports as MHTML documents | ✅ Enabled | 🟡 Medium | Users can export paginated reports as MHTML documents. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-export-sharing) |
| Export reports as Word documents | ✅ Enabled | 🟡 Medium | Users can export paginated reports as Word documents. | [Docs](https://learn.microsoft.com/en-us/power-bi/paginated-reports/report-builder/export-microsoft-word-report-builder) |
| Export reports as XML documents | ✅ Enabled | 🟡 Medium | Users can export paginated reports as XML documents. | [Docs](https://learn.microsoft.com/en-us/power-bi/paginated-reports/report-builder/export-xml-report-builder) |
| Export reports as image files | ❌ Disabled | 🟢 Low | Allows using the export report to file API to export reports as image files. | [Docs](https://learn.microsoft.com/en-us/power-bi/paginated-reports/report-builder/export-image-file-report-builder) |
| Print dashboards and reports | ✅ Enabled | 🟢 Low | Users can print dashboards and reports. | [Docs](https://learn.microsoft.com/en-us/power-bi/consumer/end-user-print) |
| Certification | ❌ Disabled | 🟠 High | Allows people to certify items as trusted sources for the organization. When certified, the certifier's contact details are visible. **Recommended to configure** with specific security groups for data governance. | [Docs](https://learn.microsoft.com/en-us/power-bi/admin/service-admin-setup-certification) |
| Endorse master data | ❌ Disabled | 🟠 High | Allows endorsing items (lakehouses, warehouses, datamarts) as core data sources for the organization's data records. | [Docs](https://learn.microsoft.com/fabric/governance/endorsement-overview) |
| Users can set up email subscriptions | ✅ Enabled | 🟡 Medium | Users can create email subscriptions to reports and dashboards for automated delivery. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/end-user-subscribe) |
| B2B guest users can set up and be subscribed to email subscriptions | ✅ Enabled | 🟠 High | B2B guest users can create and receive email subscriptions. Consider whether external users should receive scheduled report deliveries. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-export-sharing#b2b-guest-users-can-set-up-and-be-subscribed-to-email-subscriptions) |
| Users can send email subscriptions to external users | ✅ Enabled | 🟠 High | Allows sending email subscriptions to users outside your Entra ID. **Data leaves the organization via email.** Review carefully. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-export-sharing#users-can-send-email-subscriptions-to-external-users) |
| Featured content | ✅ Enabled | 🟢 Low | Users can promote published content to the Featured section of Power BI Home. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-featured-content) |
| Allow connections to featured tables | ✅ Enabled | 🟡 Medium | Allows users to access and calculate data from featured tables (defined in Power BI Desktop) via Excel's data types gallery. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-excel-featured-tables) |
| Allow shareable links to grant access to everyone in your organization | ✅ Enabled | 🟠 High | Enables "everyone in your organization" sharing links. Does not work for external users. Consider restricting to prevent oversharing. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-export-sharing#allow-shareable-links-to-grant-access-to-everyone-in-your-organization) |
| Enable Microsoft Teams integration | ✅ Enabled | 🟡 Medium | Enables Teams-related features: launching Teams experiences from Power BI, the Power BI app for Teams, and Power BI notifications in Teams. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-collaborate-microsoft-teams) |
| Install Power BI app for Microsoft Teams automatically | ✅ Enabled | 🟢 Low | Auto-installs the Power BI app for Teams when users use Fabric. Enables notifications and easier collaboration. | [Docs](https://go.microsoft.com/fwlink/?linkid=2171149) |
| Enable Power BI add-in for PowerPoint | ✅ Enabled | 🟡 Medium | Lets users embed live Power BI data into PowerPoint presentations. Requires Office admin to enable add-in support. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-export-sharing#enable-power-bi-add-in-for-powerpoint) |
| Allow DirectQuery connections to Power BI semantic models | ✅ Enabled | 🟡 Medium | Enables DirectQuery connections to existing semantic models, allowing users to build composite models on top of shared datasets. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-export-sharing#allow-directquery-connections-to-power-bi-semantic-models) |
| Guest users can work with shared semantic models in their own tenants | ❌ Disabled | 🟠 High | Allows authorized guest users to discover shared semantic models in OneLake data hub and work with them in their own Power BI tenants. **Cross-tenant data access.** | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-dataset-external-org-share-admin#allow-guest-users-to-work-with-shared-datasets-in-their-own-tenants) |
| Allow specific users to turn on external data sharing | ✅ Enabled | 🔴 Critical | Controls which users can enable the external data sharing option on semantic models that allows authorized guest users to discover, connect to, and work with shared datasets in their own tenants. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-dataset-external-org-share-admin#allow-specific-users-to-turn-on-external-data-sharing) |
| Users with read or write permission can download data from notebooks | ✅ Enabled | 🟡 Medium | Allows users with read/write permission to download or export data from tables in notebook outputs. | [Docs](https://go.microsoft.com/fwlink/?linkid=2299416) |

---

## 7. Discovery Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-discovery)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Make promoted content discoverable | ✅ Enabled | 🟡 Medium | Users who can promote content can make it discoverable by users who don't have access. Helps surface important content across the organization. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-discovery) |
| Make certified content discoverable | ✅ Enabled | 🟡 Medium | Users who can certify content can make it discoverable by users who don't have access. Drives adoption of trusted, certified data assets. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-discovery) |
| Discover content | ✅ Enabled | 🟡 Medium | Allows users to find and request access to content they don't have access to, if it was made discoverable by its owners. Promotes self-service and reduces admin requests. | [Docs](https://learn.microsoft.com/fabric/governance/onelake-catalog-overview) |

---

## 8. App Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-app)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Create template organizational apps | ❌ Disabled | 🟡 Medium | Allows users to create template apps using semantic models built on one data source in Power BI Desktop for distribution outside the organization. | [Docs](https://learn.microsoft.com/en-us/power-bi/connect-data/service-template-apps-create) |
| Push apps to end users | ✅ Enabled | 🟡 Medium | Allows report creators to share apps directly with end users without requiring installation from AppSource. Apps auto-install for target users. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-create-distribute-apps#automatically-install-apps-for-end-users) |
| Publish apps to the entire organization | ✅ Enabled | 🟠 High | Allows users to publish apps to everyone in the organization rather than specific groups. Consider restricting to prevent oversharing. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-create-distribute-apps#publish-the-app-to-your-entire-organization) |

---

## 9. Integration Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-integration)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Allow XMLA endpoints and Analyze in Excel with on-premises semantic models | ✅ Enabled | 🟠 High | Enables Excel interaction with on-premises Power BI semantic models and connections to XMLA endpoints. XMLA endpoints are essential for enterprise tooling and ALM. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-analyze-in-excel) |
| Semantic Model Execute Queries REST API | ✅ Enabled | 🟡 Medium | Allows querying semantic models using DAX through Power BI REST APIs. Used by developers and automated processes. | [Docs](https://learn.microsoft.com/en-us/rest/api/power-bi/datasets/execute-queries) |
| Users can use the Power BI MCP server endpoint (preview) | ✅ Enabled for a subset | 🟡 Medium | Enables the Power BI Model Context Protocol (MCP) server endpoint for connecting MCP clients (like GitHub Copilot in VS Code) to interact with Power BI artifacts. Service principals need "Service principals can call Fabric public APIs" enabled. | [Docs](https://go.microsoft.com/fwlink/?linkid=2338916) |
| Use ArcGIS Maps for Power BI | ✅ Enabled | 🟢 Low | Allows using the ArcGIS Maps visualization provided by Esri in Power BI reports. | [Docs](https://learn.microsoft.com/en-us/power-bi/visuals/power-bi-visualizations-arcgis) |
| Use global search for Power BI | ✅ Enabled | 🟢 Low | Enables the global search bar at the top of the Power BI portal page. | [Docs](https://learn.microsoft.com/en-us/power-bi/consumer/end-user-search-sort) |
| Users can use the Azure Maps visual | ✅ Enabled | 🟢 Low | Enables the Azure Maps visual. Data may be temporarily stored by Microsoft for location translation. Subject to Azure Maps Terms of Use. | [Docs](https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-get-started) |
| Data sent to Azure Maps can be processed outside your tenant's geographic region | ❌ Disabled | 🟠 High | When enabled, Azure Maps data may be processed outside your tenant's geographic region or compliance boundary. **Data residency implications.** | [Docs](https://go.microsoft.com/fwlink/?linkid=2289253) |
| Data sent to Azure Maps can be processed by Microsoft Online Services Subprocessors | ❌ Disabled | 🟠 High | Allows Azure Maps subprocessors to process mapping data. Data may be stored/processed in the US or other countries. **Data residency implications.** | [Docs](https://go.microsoft.com/fwlink/?linkid=2289928) |
| Map and filled map visuals | ❌ Disabled | 🟢 Low | Enables the map and filled map visualizations in reports. | [Docs](https://learn.microsoft.com/en-us/power-bi/visuals/power-bi-visualization-filled-maps-choropleths) |
| Integration with SharePoint and Microsoft Lists | ✅ Enabled | 🟡 Medium | Users can launch Power BI from SharePoint lists and Microsoft Lists, build reports, and publish them back. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-integration#integration-with-sharepoint-and-microsoft-lists) |
| Dremio SSO | ❌ Disabled | 🟡 Medium | Enables SSO for Dremio. User access token information (name, email) is sent to Dremio for authentication. | [Docs](https://powerquery.microsoft.com/blog/azure-ad-based-single-sign-on-for-dremio-cloud-and-power-bi) |
| Snowflake SSO | ❌ Disabled | 🟡 Medium | Enables SSO for Snowflake. User access token information (name, email) is sent to Snowflake for authentication. | [Docs](https://learn.microsoft.com/en-us/power-bi/connect-data/service-connect-snowflake) |
| Redshift SSO | ❌ Disabled | 🟡 Medium | Enables SSO for Redshift. User access token information (name, email) is sent to Redshift for authentication. | [Docs](https://learn.microsoft.com/en-us/power-bi/connect-data/service-gateway-sso-overview) |
| Google BigQuery SSO | ❌ Disabled | 🟡 Medium | Enables SSO for Google BigQuery. User access token information (name, email) is sent to Google BigQuery for authentication. | [Docs](https://learn.microsoft.com/en-us/power-query/connectors/google-bigquery-aad) |
| Microsoft Entra single sign-on for data gateway | ❌ Disabled | 🟡 Medium | Enables Entra ID SSO for on-premises data gateways. User access token information (name, email) is sent to data sources. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-integration#azure-ad-single-sign-on-sso-for-gateway) |
| Users can view Power BI files saved in OneDrive and SharePoint (preview) | ✅ Enabled | 🟡 Medium | Enables viewing Power BI files (.pbix) saved in OneDrive for Business or SharePoint directly. Permissions are controlled by OneDrive/SharePoint. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-sharepoint-viewer) |
| Users can share links to Power BI files in OneDrive and SharePoint through Desktop (preview) | ✅ Enabled | 🟡 Medium | Allows sharing links to .pbix files stored in OneDrive/SharePoint directly from Power BI Desktop. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-integration#users-can-share-links-to-power-bi-files-stored-in-onedrive-and-sharepoint-through-power-bi-desktop-preview) |
| Enable granular access control for all data connections | ❌ Disabled | 🟠 High | Enforces strict access control for all data connection types. Shared items are disconnected from data sources if edited by users without connection permission. | [Docs](https://go.microsoft.com/fwlink/?linkid=2226159) |
| Semantic models can export data to OneLake | ✅ Enabled | 🟡 Medium | Allows semantic models configured for OneLake integration to send import tables to OneLake for use in lakehouses and warehouses. | [Docs](https://learn.microsoft.com/en-us/power-bi/enterprise/onelake-integration-overview) |
| Semantic model owners can choose to automatically update from OneDrive or SharePoint | ✅ Enabled | 🟡 Medium | Allows semantic model owners to enable automatic updates when the corresponding .pbix file in OneDrive/SharePoint changes. | [Docs](https://learn.microsoft.com/en-us/power-bi/connect-data/refresh-desktop-file-onedrive) |
| ArcGIS GeoAnalytics for Fabric Runtime | ❌ Disabled | 🟢 Low | Enables Esri's ArcGIS GeoAnalytics in the Fabric Spark Runtime for spatial SQL functions and analysis on big data. | [Docs](https://go.microsoft.com/fwlink/?linkid=2281344) |
| Allow non-Entra ID auth in Eventstream | ✅ Enabled | 🟡 Medium | Controls whether key-based authentication is allowed in Eventstream's Custom Endpoint. Disabling ensures only Entra ID authentication is used, reducing unauthorized access risk. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-integration) |

---

## 10. Power BI Visuals

[Category Documentation](https://learn.microsoft.com/en-us/power-bi/admin/organizational-visuals)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Allow visuals created using the Power BI SDK | ✅ Enabled | 🟡 Medium | Allows users to add, view, share, and interact with custom visuals from AppSource or file uploads. Organizational visuals page is not affected. | [Docs](https://learn.microsoft.com/en-us/power-bi/admin/organizational-visuals#visuals-from-appsource-or-a-file) |
| Add and use certified visuals only (block uncertified) | ❌ Disabled | 🟠 High | Restricts custom visuals to certified-only. When enabled, uncertified visuals cannot be added. **Consider enabling for security-sensitive environments** — uncertified visuals may execute arbitrary code. | [Docs](https://learn.microsoft.com/en-us/power-bi/admin/organizational-visuals#certified-power-bi-visuals) |
| Allow downloads from custom visuals | ❌ Disabled | 🟠 High | Allows custom visuals to download any information available to the visual (summarized data, visual configuration) with user consent. **Data exfiltration risk** — not affected by Export/Sharing settings. | [Docs](https://learn.microsoft.com/en-us/power-bi/admin/organizational-visuals#export-data-to-file) |
| AppSource Custom Visuals SSO | ❌ Disabled | 🟡 Medium | Enables SSO for AppSource custom visuals, allowing them to get Entra ID access tokens. User names and emails may be sent across regions and compliance boundaries. | [Docs](https://learn.microsoft.com/fabric/admin/organizational-visuals#appsource-custom-visuals-sso) |
| Allow access to the browser's local storage | ✅ Enabled | 🟡 Medium | Allows custom visuals to store information in the user's browser local storage. | [Docs](https://learn.microsoft.com/en-us/power-bi/admin/organizational-visuals) |

---

## 11. R and Python Visuals Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-r-python-visuals)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Interact with and share R and Python visuals | ✅ Enabled | 🟡 Medium | Allows users to interact with and share visuals created with R or Python scripts. Note: R and Python visuals execute code, so consider the security implications. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-r-python-visuals#interact-with-and-share-r-and-python-visuals) |

---

## 12. Audit and Usage Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-audit-usage)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Usage metrics for content creators | ✅ Enabled | 🟡 Medium | Allows content creators to see usage metrics for dashboards, reports, and semantic models they have permissions to. Informs content optimization decisions. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-modern-usage-metrics) |
| Per-user data in usage metrics for content creators | ✅ Enabled | 🟠 High | Exposes display names and email addresses of users in usage metrics. **Privacy implication** — consider disabling if per-user tracking is a concern. | [Docs](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-modern-usage-metrics#exclude-user-information-from-usage-metrics-reports) |
| Show user data in the Fabric Capacity Metrics app and reports | ✅ Enabled | 🟡 Medium | Displays active user data (names, emails) in the Capacity Metrics app. Useful for capacity planning and chargeback. | [Docs](https://learn.microsoft.com/en-us/fabric/enterprise/metrics-app) |
| Azure Log Analytics connections for workspace administrators | ❌ Disabled | 🟡 Medium | Allows workspace admins to send Power BI diagnostic data to Azure Log Analytics for advanced monitoring and analysis. | [Docs](https://learn.microsoft.com/en-us/power-bi/transform-model/log-analytics/desktop-log-analytics-configure) |
| Workspace admins can turn on monitoring for their workspaces (preview) | ✅ Enabled | 🟡 Medium | Allows workspace admins to enable monitoring, which creates a read-only Eventhouse with KQL database for logging workspace activity. | [Docs](https://learn.microsoft.com/fabric/fundamentals/enable-workspace-monitoring) |
| Microsoft can store query text to aid in support investigations | ✅ Enabled | 🟡 Medium | Securely stores query text for some items (e.g., semantic models) to aid Microsoft support investigations. Disabling may reduce Microsoft's ability to provide support. | [Docs](https://learn.microsoft.com/fabric/admin/query-text-storage) |

---

## 13. Dashboard Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-dashboard)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Web content on dashboard tiles | ❌ Disabled | 🔴 Critical | Allows adding and viewing web content tiles on dashboards. ⚠️ **Security risk:** may expose the organization to malicious web content via iframes. **Recommended to keep disabled** unless there is a specific business need. | [Docs](https://learn.microsoft.com/en-us/power-bi/create-reports/service-dashboard-add-widget#add-web-content) |

---

## 14. Developer Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-developer)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Embed content in apps | ✅ Enabled | 🟠 High | Allows embedding Power BI dashboards and reports in web applications using the "Embed for your customers" method. Essential for ISV and custom app scenarios. | [Docs](https://learn.microsoft.com/en-us/power-bi/developer/embedded/embedded-analytics-power-bi) |
| Service principals can create workspaces, connections, and deployment pipelines | ✅ Enabled for a subset | 🟠 High | Allows service principals to create workspaces, connections, and deployment pipelines. Should be scoped to specific security groups for CI/CD automation. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-developer#service-principals-can-create-workspaces-connections-and-deployment-pipelines) |
| Service principals can call Fabric public APIs | ✅ Enabled | 🟠 High | Allows service principals with appropriate roles/permissions to call Fabric public APIs. Core requirement for automated administration and CI/CD. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-developer#service-principals-can-call-fabric-public-apis) |
| Allow service principals to create and use profiles | ❌ Disabled | 🟡 Medium | Allows service principals to create and use profiles for multi-tenant embedded analytics scenarios. | [Docs](https://learn.microsoft.com/en-us/power-bi/developer/embedded/embed-multi-tenancy) |
| Block ResourceKey Authentication | ❌ Disabled | 🟠 High | Blocks resource key-based authentication for streaming semantic models API. **Consider enabling** for improved security — resource keys are less secure than Entra ID auth. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-developer#block-resourcekey-authentication) |
| Define maximum number of Fabric identities in a tenant | ❌ Disabled | 🟡 Medium | Allows admins to cap the number of Fabric identities in the tenant. Default limit is 10,000 if disabled. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-developer#define-maximum-number-of-fabric-identities-in-a-tenant) |

---

## 15. Admin API Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-admin-api-settings)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Service principals can access read-only admin APIs | ✅ Enabled for a subset | 🟠 High | Allows service principals in specified security groups to authenticate to read-only admin APIs. Provides read-only access to all admin API information (including user names, emails, and metadata). | [Docs](https://learn.microsoft.com/en-us/power-bi/enterprise/read-only-apis-service-principal-authentication) |
| Service principals can access admin APIs used for updates | ✅ Enabled for a subset | 🔴 Critical | Allows service principals to authenticate to admin APIs for **write operations**. Provides full access including user names, emails, and metadata. **Ensure only trusted service principals are in the allowed group.** | [Docs](https://learn.microsoft.com/en-us/fabric/admin/enable-service-principal-admin-apis) |
| Enhance admin APIs responses with detailed metadata | ✅ Enabled for a subset | 🟡 Medium | Returns detailed metadata (table/column names) in admin API responses like GetScanResult. Required for metadata scanning and governance tooling. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-admin-api-settings#enhance-admin-apis-responses-with-detailed-metadata) |
| Enhance admin APIs responses with DAX and mashup expressions | ✅ Enabled for a subset | 🟠 High | Returns DAX and mashup (Power Query) expressions in admin API responses. **Exposes business logic and connection strings** — restrict to trusted security groups. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-admin-api-settings#enhance-admin-apis-responses-with-dax-and-mashup-expressions) |

---

## 16. Gen1 Dataflow Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-dataflow)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Create and use Gen1 dataflows | ✅ Enabled | 🟡 Medium | Allows users to create and use Gen1 dataflows (Power Query Online). Gen2 dataflows are preferred for new development, but Gen1 may still be needed for backward compatibility. | [Docs](https://learn.microsoft.com/en-us/power-bi/transform-model/dataflows/dataflows-introduction-self-service) |

---

## 17. Template App Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-template-app)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Publish template apps | ✅ Enabled | 🟡 Medium | Allows publishing template apps for distribution to clients outside the organization. | [Docs](https://learn.microsoft.com/en-us/power-bi/connect-data/service-template-apps-overview) |
| Install template apps | ✅ Enabled | 🟡 Medium | Users can install template apps from outside the organization. Installing creates an upgraded workspace. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-template-app#install-template-apps) |
| Install template apps not listed in AppSource | ❌ Disabled | 🟠 High | Allows installing template apps that weren't published to Microsoft AppSource. **Security consideration:** unvetted apps could contain malicious content. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-template-app) |

---

## 18. Q&A Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-qa)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Review questions | ✅ Enabled | 🟢 Low | Allows semantic model owners to review questions people asked about their data using the Q&A natural language feature. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-qa) |
| Synonym sharing | ✅ Enabled | 🟢 Low | Allows sharing Q&A synonyms with the organization to improve natural language query results. | [Docs](https://learn.microsoft.com/en-us/power-bi/natural-language/q-and-a-tooling-intro#field-synonyms) |

---

## 19. Explore Settings (Preview)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Users with view permission can launch Explore | ✅ Enabled | 🟢 Low | Enables the lightweight visual data exploration experience (Explore) for users with view permission on a semantic model or connected items. Allows quick ad hoc analysis. | [Docs](https://learn.microsoft.com/en-us/power-bi/consumer/explore-data-service#permissions-and-requirements) |

---

## 20. Semantic Model Security

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Block republish and disable package refresh | ❌ Disabled | 🟠 High | When enabled, only the semantic model owner can publish updates, and package refresh is disabled. **Consider enabling** if you need strict control over who can modify production semantic models. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-dataset-security#block-republish-and-disable-package-refresh) |

---

## 21. Advanced Networking

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-advanced-networking)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Tenant-level Private Link | ❌ Disabled | 🔴 Critical | Enables Azure Private Link for secure access to your Fabric tenant over a private network. Requires Azure setup. **Key security control for enterprise environments** that require private connectivity. | [Docs](https://learn.microsoft.com/fabric/security/security-private-links-overview) |
| Block Public Internet Access | ❌ Disabled | 🔴 Critical | Blocks access to Fabric via the public internet. Only Private Link connections can reach the tenant. Takes 10–20 minutes to take effect. **Only enable after Private Link is fully configured**, or you will lock out all users. | [Docs](https://learn.microsoft.com/en-us/power-bi/enterprise/service-security-private-links) |
| Configure workspace-level inbound network rules | ❌ Disabled | 🟠 High | Allows workspace admins to configure inbound private link access per workspace. Existing tenant-level private links will NOT connect to configured workspaces. | [Docs](https://go.microsoft.com/fwlink/?linkid=2272575) |
| Configure workspace-level outbound network rules | ❌ Disabled | 🟠 High | Allows workspace admins to configure outbound access protection per workspace. Turning off this tenant setting reverts all workspaces. | [Docs](https://go.microsoft.com/fwlink/?linkid=2310620) |
| Configure workspace-level IP firewall rules and trusted resource instances (preview) | ✅ Enabled | 🟠 High | Allows workspace admins to set IP firewall rules and trusted resource instances, even when tenant-level public access is blocked. Provides granular network control. | [Docs](https://go.microsoft.com/fwlink/?linkid=2223103) |

---

## 22. Encryption

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Apply customer-managed keys | ❌ Disabled | 🔴 Critical | Enables workspace-level encryption using customer-managed keys (CMK). When off, data is encrypted with Microsoft-managed keys. **Required for organizations with strict key management or regulatory requirements** (e.g., financial services, healthcare). | [Docs](https://go.microsoft.com/fwlink/?linkid=2308801) |

---

## 23. Scorecards Settings

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Create and use Scorecards | ✅ Enabled | 🟢 Low | Allows creation and use of Scorecards (metrics/goals tracking) in Power BI. | [Docs](https://learn.microsoft.com/en-us/power-bi/create-reports/service-goals-introduction) |

---

## 24. User Experience Experiments

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-user-experience-experiments)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Help Power BI optimize your experience | ✅ Enabled | 🟢 Low | Users receive minor UX variations (content, layout, design) that Power BI is experimenting with before general release. Consider disabling if consistent UI is important. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-user-experience-experiments) |

---

## 25. Share Data with Your Microsoft 365 Services

[Category Documentation](https://learn.microsoft.com/fabric/admin/admin-share-power-bi-metadata-microsoft-365-services)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Share Fabric data with your Microsoft 365 services | ✅ Enabled | 🟠 High | Allows Fabric data (report titles, chart labels, sharing history, etc.) to be used by M365 services for search results and recommendations. Users only see content they have access to. Auto-enabled when Fabric and M365 tenants are in the same geography. | [Docs](https://learn.microsoft.com/fabric/admin/admin-share-power-bi-metadata-microsoft-365-services) |

---

## 26. Insights Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-insights)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Receive notifications for top insights (preview) | ✅ Enabled | 🟢 Low | Users can enable notifications for top insights in report settings. | [Docs](https://learn.microsoft.com/en-us/power-bi/create-reports/insights) |
| Show entry points for insights (preview) | ✅ Enabled | 🟢 Low | Shows entry points within reports for requesting AI-generated insights. | [Docs](https://learn.microsoft.com/en-us/power-bi/create-reports/insights) |

---

## 27. Datamart Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-datamart)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Create Datamarts (preview) | ✅ Enabled | 🟡 Medium | Allows creation of Datamarts — self-service databases with auto-generated semantic models. Preview feature; consider whether your organization needs this alongside Warehouses and Lakehouses. | [Docs](https://learn.microsoft.com/en-us/power-bi/transform-model/datamarts/datamarts-administration) |

---

## 28. Semantic Model Settings

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Users can edit semantic models in the Power BI service | ✅ Enabled | 🟡 Medium | Allows editing semantic models directly in the web service. Does not apply to DirectLake semantic models or editing via API/XMLA endpoints. | [Docs](https://go.microsoft.com/fwlink/?linkid=2227332) |

---

## 29. Scale-out Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-scale-out)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Scale out queries for large semantic models | ✅ Enabled | 🟡 Medium | Automatically distributes queries across additional semantic model replicas when query volume is high (Premium feature, requires large semantic model storage format). | [Docs](https://learn.microsoft.com/en-us/power-bi/enterprise/service-premium-scale-out) |

---

## 30. OneLake Settings

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-onelake)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Users can access data stored in OneLake with apps external to Fabric | ✅ Enabled | 🔴 Critical | Allows external applications (custom apps using ADLS APIs, OneLake File Explorer, Databricks) to access data in OneLake. **Controls the data perimeter** — review whether external app access is required. | [Docs](https://learn.microsoft.com/fabric/onelake/security/fabric-onelake-security#allow-apps-running-outside-of-fabric-to-access-data-via-onelake) |
| Use short-lived user-delegated SAS tokens | ✅ Enabled | 🟠 High | Enables applications to access OneLake data through short-lived SAS tokens based on a user's Entra identity. Tokens cannot exceed 1-hour lifetime. | [Docs](https://go.microsoft.com/fwlink/?linkid=2268260) |
| Authenticate with OneLake user-delegated SAS tokens | ❌ Disabled | 🟠 High | Allows applications to authenticate using OneLake SAS tokens generated by requesting a user delegation key. Requires "Use short-lived user-delegated SAS tokens" to be on. | [Docs](https://go.microsoft.com/fwlink/?linkid=2268260) |
| Users can sync data in OneLake with the OneLake File Explorer app | ✅ Enabled | 🟡 Medium | Enables OneLake File Explorer, which syncs OneLake items to Windows File Explorer (similar to OneDrive). | [Docs](https://learn.microsoft.com/fabric/onelake/onelake-file-explorer) |
| Include end-user identifiers in OneLake diagnostic logs | ✅ Enabled | 🟡 Medium | Captures email addresses and IP addresses in OneLake diagnostic logs. Useful for diagnostics and investigations. When disabled, these fields are redacted. | [Docs](https://go.microsoft.com/fwlink/?linkid=2335502) |

---

## 31. Git Integration

[Category Documentation](https://learn.microsoft.com/fabric/admin/git-integration-admin-settings)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Users can synchronize workspace items with their Git repositories | ✅ Enabled | 🟠 High | Enables Git integration for importing/exporting workspace items for collaboration and version control. Core feature for CI/CD and ALM workflows. | [Docs](https://learn.microsoft.com/fabric/cicd/git-integration/intro-to-git-integration) |
| Users can export items to Git repositories in other geographical locations | ❌ Disabled | 🟠 High | When workspace and Git repo are in different geographies, this must be enabled. **Data residency implication** — content may leave your geographic boundary. | [Docs](https://learn.microsoft.com/fabric/admin/git-integration-admin-settings#users-can-export-items-to-git-repositories-in-other-geographical-locations) |
| Users can export workspace items with applied sensitivity labels to Git repositories | ✅ Enabled | 🟠 High | Allows exporting items with sensitivity labels to Git. Consider whether labeled/classified content should be stored in external repositories. | [Docs](https://learn.microsoft.com/fabric/admin/git-integration-admin-settings#users-can-export-workspace-items-with-applied-sensitivity-labels-to-git-repositories) |
| Users can sync workspace items with GitHub repositories | ❌ Disabled | 🟡 Medium | Enables GitHub as a Git provider (in addition to Azure DevOps). Enable if your organization uses GitHub for source control. | [Docs](https://learn.microsoft.com/fabric/admin/git-integration-admin-settings#users-can-sync-workspace-items-with-github-repositories) |

---

## 32. Copilot and Azure OpenAI Service

[Category Documentation](https://learn.microsoft.com/fabric/admin/service-admin-portal-copilot)

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Users can use Copilot and other features powered by Azure OpenAI | ✅ Enabled | 🔴 Critical | Master switch for all Copilot and AI features in Fabric. Agreeing enables all AI features including those in preview. Can be managed at tenant and capacity levels. EU Data Boundary commitments apply for EU customers. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-copilot) |
| Users can access a standalone, cross-item Power BI Copilot experience (preview) | ✅ Enabled | 🟡 Medium | Provides a dedicated Copilot tab in Power BI navigation for finding, analyzing, and discussing Fabric items. Requires the main Copilot setting above. | [Docs](https://go.microsoft.com/fwlink/?linkid=2306434) |
| Data sent to Azure OpenAI can be processed outside your capacity's geographic region | ❌ Disabled | 🔴 Critical | Allows Copilot/AI data to be processed outside your capacity's geographic region. **Only required if your capacity is outside the EU Data Boundary or US.** Significant data residency and compliance implication. | [Docs](https://learn.microsoft.com/en-us/azure/azure-maps/geographic-scope) |
| Capacities can be designated as Fabric Copilot capacities | ✅ Enabled | 🟡 Medium | Allows capacity admins to designate capacities as Copilot capacities to consolidate Copilot usage and billing. Capacity admins can see item names associated with Copilot activity. | [Docs](https://learn.microsoft.com/fabric/admin/service-admin-portal-copilot#capacities-can-be-designated-as-fabric-copilot-capacities) |
| Data sent to Azure OpenAI can be stored outside your capacity's geographic region | ❌ Disabled | 🔴 Critical | Allows Copilot/AI data to be **stored** (not just processed) outside your capacity's geographic region. **Most restrictive data residency setting** for AI features. | [Docs](https://learn.microsoft.com/en-us/fabric/fundamentals/copilot-fabric-overview#available-regions) |
| Only show approved items in the standalone Copilot experience (preview) | ❌ Disabled | 🟡 Medium | Restricts standalone Copilot to only show apps, data agents, and items marked "approved for Copilot." Users can still manually attach items. | [Docs](https://go.microsoft.com/fwlink/?linkid=2311781) |

---

## 33. Azure Maps Services

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Users can use Azure Maps services | ✅ Enabled | 🟢 Low | Enables features powered by Azure Maps services. Subject to Azure Maps Terms of Use. EU Data Boundary commitments apply for EU customers. | [Docs](https://go.microsoft.com/fwlink/?linkid=2310613) |
| Data sent to Azure Maps can be processed outside your capacity's geographic region | ❌ Disabled | 🟠 High | Allows Azure Maps data to be processed outside your capacity's geographic region where the service is available. **Data residency implication.** | [Docs](https://go.microsoft.com/fwlink/?linkid=2289253) |
| Users can use Azure Maps Weather Services (Preview) | ✅ Enabled | 🟢 Low | Enables access to weather data from Azure Maps Weather, sourced from AccuWeather. | [Docs](https://go.microsoft.com/fwlink/?linkid=2340279) |

---

## 34. Additional Workloads

| Setting | Current Value | Priority | Description | Learn More |
|---------|--------------|----------|-------------|------------|
| Workspace admins can add and remove additional workloads (preview) | ✅ Enabled | 🟡 Medium | Allows workspace admins to add/remove workloads. When users interact with workloads, data and access tokens (name, email) are sent to the publisher. Sensitivity labels and encryption are NOT applied to workload items. | [Docs](https://go.microsoft.com/fwlink/?linkid=2268082) |
| Capacity admins and contributors can add and remove additional workloads | ✅ Enabled | 🟡 Medium | Allows capacity admins or contributors to add/remove workloads at the capacity level. Same data sharing and sensitivity label considerations as above. | [Docs](https://learn.microsoft.com/fabric/workload-development-kit/environment-setup#enable-the-development-tenant-setting) |
| Workspace admins can develop partner workloads | ❌ Disabled | 🟢 Low | Allows workspace admins to develop partner workloads using a local machine development environment. Only needed for workload developers. | [Docs](https://learn.microsoft.com/en-us/fabric/workload-development-kit/environment-setup#enable-the-development-tenant-setting) |
| Users can see and work with additional workloads not validated by Microsoft | ❌ Disabled | 🟠 High | Allows users to see and work with unvalidated workloads. **Only enable workloads from publishers you trust** to meet organizational policies. | [Docs](https://learn.microsoft.com/fabric/workload-development-kit/publish-workload-requirements) |

---

## Summary: Settings by Priority

### 🔴 Critical (Must Review)

| # | Setting | Current Value |
|---|---------|--------------|
| 1 | Users can create Fabric items | Enabled for a subset |
| 2 | Create workspaces | Enabled for a subset |
| 3 | Allow users to apply sensitivity labels for content | Disabled |
| 4 | Apply sensitivity labels from data sources | Disabled |
| 5 | Restrict content with protected labels from being shared via link | Disabled |
| 6 | External data sharing | Disabled |
| 7 | Users can accept external data shares | Disabled |
| 8 | Guest users can access Microsoft Fabric | Enabled |
| 9 | Users can invite guest users to collaborate | Enabled |
| 10 | Publish to web | Enabled |
| 11 | Allow specific users to turn on external data sharing | Enabled |
| 12 | Web content on dashboard tiles | Disabled |
| 13 | Service principals can access admin APIs used for updates | Enabled for a subset |
| 14 | Tenant-level Private Link | Disabled |
| 15 | Block Public Internet Access | Disabled |
| 16 | Apply customer-managed keys | Disabled |
| 17 | Users can access data stored in OneLake with apps external to Fabric | Enabled |
| 18 | Users can use Copilot and other features powered by Azure OpenAI | Enabled |
| 19 | Data sent to Azure OpenAI can be processed outside your capacity's geographic region | Disabled |
| 20 | Data sent to Azure OpenAI can be stored outside your capacity's geographic region | Disabled |

### 🟠 High (Important to Discuss)

| # | Setting | Current Value |
|---|---------|--------------|
| 1 | Receive email and Teams notifications for service outages | Disabled |
| 2 | Users can try Microsoft Fabric paid features | Enabled |
| 3 | Use semantic models across workspaces | Enabled |
| 4 | Define workspace retention period | Enabled |
| 5 | Fabric item recovery | Disabled |
| 6 | Automatically apply sensitivity labels to downstream content | Disabled |
| 7 | Allow Microsoft Purview to secure AI interactions | Enabled |
| 8 | Guest users can browse and access Fabric content | Disabled |
| 9 | Certification | Disabled |
| 10 | Endorse master data | Disabled |
| 11 | B2B guest users can set up and be subscribed to email subscriptions | Enabled |
| 12 | Users can send email subscriptions to external users | Enabled |
| 13 | Allow shareable links to grant access to everyone in your organization | Enabled |
| 14 | Guest users can work with shared semantic models in their own tenants | Disabled |
| 15 | Publish apps to the entire organization | Enabled |
| 16 | Allow XMLA endpoints and Analyze in Excel with on-premises semantic models | Enabled |
| 17 | Data sent to Azure Maps can be processed outside your tenant's geographic region | Disabled |
| 18 | Data sent to Azure Maps can be processed by Microsoft Online Services Subprocessors | Disabled |
| 19 | Enable granular access control for all data connections | Disabled |
| 20 | Add and use certified visuals only (block uncertified) | Disabled |
| 21 | Allow downloads from custom visuals | Disabled |
| 22 | Per-user data in usage metrics for content creators | Enabled |
| 23 | Embed content in apps | Enabled |
| 24 | Service principals can create workspaces, connections, and deployment pipelines | Enabled for a subset |
| 25 | Service principals can call Fabric public APIs | Enabled |
| 26 | Block ResourceKey Authentication | Disabled |
| 27 | Service principals can access read-only admin APIs | Enabled for a subset |
| 28 | Enhance admin APIs responses with DAX and mashup expressions | Enabled for a subset |
| 29 | Install template apps not listed in AppSource | Disabled |
| 30 | Block republish and disable package refresh | Disabled |
| 31 | Configure workspace-level inbound network rules | Disabled |
| 32 | Configure workspace-level outbound network rules | Disabled |
| 33 | Configure workspace-level IP firewall rules | Enabled |
| 34 | Use short-lived user-delegated SAS tokens | Enabled |
| 35 | Authenticate with OneLake user-delegated SAS tokens | Disabled |
| 36 | Users can synchronize workspace items with their Git repositories | Enabled |
| 37 | Users can export items to Git repos in other geographical locations | Disabled |
| 38 | Users can export workspace items with applied sensitivity labels to Git | Enabled |
| 39 | Share Fabric data with your Microsoft 365 services | Enabled |
| 40 | Data sent to Azure Maps can be processed outside capacity's geographic region | Disabled |
| 41 | Users can see and work with unvalidated workloads | Disabled |

---

## Key Recommendations for Discussion

1. **🔴 Publish to Web** — Currently enabled for the entire organization. This allows anyone to create publicly accessible reports with no authentication. Consider disabling or restricting to a security group.

2. **🔴 Information Protection** — All sensitivity label settings are disabled. If your organization uses Microsoft Purview, enabling these settings provides critical data classification and protection capabilities.

3. **🟠 Fabric Item Recovery** — Currently disabled. Strongly recommend enabling with a 30–90 day retention period to protect against accidental deletions.

4. **🟠 Service Outage Notifications** — Currently disabled. Enable and point to your admin/operations security group so the team is immediately notified of incidents.

5. **🟠 Certification & Endorsement** — Both disabled. These are key governance features for establishing trusted, official data assets across the organization.

6. **🟠 External Email Subscriptions** — Currently enabled for the entire organization. Data can leave the organization via scheduled email reports. Consider restricting.

7. **🔴 Advanced Networking** — Private Link and public access blocking are disabled. Evaluate whether your security requirements mandate private connectivity to Fabric.

8. **🔴 OneLake External Access** — Enabled for the entire organization. External applications can access OneLake data. Review whether this aligns with your data perimeter strategy.

---

*Document generated from tenant settings as of April 14, 2026. For the most current documentation, visit the [Fabric Tenant Settings Index](https://learn.microsoft.com/fabric/admin/tenant-settings-index).*
