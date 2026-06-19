# Unified Reporting Workspace Data Sharing Process

## Architecture Diagram

:::mermaid
graph TB
    subgraph "Agency(x) Workspace"
        AGX_LH["CA_{AGENCY}_Agency_Lakehouse"]
        AGX_SCHEMAS["operational schemas<br/>(raw base tables —<br/>never shared)"]
        AGX_URL["url schema<br/>(prescribed Materialized Lake Views,<br/>agency-prefixed keys)"]
        AGX_LH --> AGX_SCHEMAS
        AGX_SCHEMAS -. "MLV refresh job<br/>(agency identity reads<br/>base tables, writes Delta)" .-> AGX_URL
    end

    subgraph "DIF [PRD] Workspace"
        DIF_SP{{"DIF Workspace Identity<br/>(Single Curator Service Principal)"}}
        CUSTOM_NB["Collect Notebook<br/>(Read → Union → Transform → Write)"]
        UR_LH["Unified Reporting Lakehouse"]
        UR_SCHEMA["interagency_sharing schema<br/>(public consumption —<br/>all agencies can read)"]
        DIF_SP --> CUSTOM_NB
        CUSTOM_NB -- "Writes consolidated<br/>multi-agency data" --> UR_LH
        UR_LH --> UR_SCHEMA
    end

    %% DIF reads the agency url MLVs by OneLake path, governed by a scoped OneLake role
    AGX_URL -- "DIF SP reads MLVs by OneLake path<br/>(OneLake Read role scoped to url;<br/>base tables denied — 403)" --> CUSTOM_NB

    %% Agencies consume from interagency_sharing
    UR_SCHEMA -. "OneLake Read role on interagency_sharing<br/>→ reader umbrella group<br/>(nested agency steward/developer groups)" .-> AGX_LH

    style DIF_SP fill:#f9a825,stroke:#f57f17,color:#000
    style CUSTOM_NB fill:#42a5f5,stroke:#1565c0,color:#fff
    style UR_SCHEMA fill:#66bb6a,stroke:#2e7d32,color:#fff
    style AGX_URL fill:#ce93d8,stroke:#6a1b9a
:::

## Narrative Description

### 1. Agency Data Preparation (Per-Agency Workspace)

Each participating agency maintains a dedicated **Agency Lakehouse** (e.g., `CA_ORC_Agency_Lakehouse`) within their own Fabric workspace. Raw operational data may reside across any number of schemas within the lakehouse — the `url` schema is not tied to any single source schema. A prescribed set of **Materialized Lake Views (MLVs) in the `url` schema** exposes a standardized, agency-prefixed version of whichever underlying data the agency elects to share.

These are **materialized** views, not logical views — this is deliberate. An MLV persists its result as a Delta table, so a consumer reads the materialized snapshot and never touches the underlying base tables. Only the agency-owned **MLV refresh job** reads the base tables (under the agency's own identity), on whatever cadence the agency schedules. Consumers therefore see a point-in-time snapshot; refresh frequency governs data freshness.

The view definitions are governed by the PSDE Steering Committee — every agency implements the same contract (same column names, types, and structure). All primary keys and foreign keys are prefixed with the agency acronym (e.g., `CustomerID = 'ORC42'`), and an `AgencyCode` column is included. This ensures that when records are unioned across agencies, every row is globally unique and traceable to its source. Any operational schema in the lakehouse is a candidate for this sharing process; only the published `url` MLVs are ever exposed.

### 2. DIF-as-Curator (Hub-and-Spoke Mediation)

Within the **DIF [PRD] Workspace**, a single **DIF Workspace Identity** (a service principal) is granted scoped read access to each agency's `url` schema. Because DIF acts as the central curator, **each agency configures access exactly once — to DIF — rather than sharing data directly with every other agency.** Onboarding the *N*-th agency adds one grant, so the number of access grants grows in step with the number of agencies, instead of with the number of agency *pairs* as it would if every agency shared peer-to-peer.

#### Access configuration (per agency lakehouse)

Least privilege here is precise, and the steps matter — broader roles silently defeat it:

1. **Entra group membership** — the DIF Workspace Identity is a member of an Entra security group (e.g., `psde_dif_dataengineers`); all grants below target the group, not the principal directly.
2. **Item-level Read share** on the agency lakehouse — *only* "Read" (connect/discover). **Not** Viewer, **not** "Read all data," **not** Contributor/Member (Contributor and above bypass OneLake security entirely).
3. **OneLake security role** — a `Grant` / `Read` role scoped to the **`url` schema only**, with the Entra group as a member.
4. **DefaultReader** — the group must be kept **out** of the lakehouse's `DefaultReader` role, which otherwise re-grants full read.

The net effect, verified in testing: the DIF SP reads `url.*` and is denied (`403 / AccessDenied`) on the base-table schemas. OneLake security enforces this at the **storage layer**, so it holds for Spark, the SQL analytics endpoint, and any other engine.

#### Collection notebook

The Service Principal runs the **Collect Notebook** (`NB_Collect_URL_Shared_Views`) that:

1. Reads each agency's `url` MLVs **by OneLake path** (see Implementation Notes), governed by the scoped OneLake role
2. Adds source-traceability columns (`_source_agency_code`, `_source_lakehouse`, `_collected_at_utc`)
3. Performs `UNION` (by name) across all participating agencies
4. Applies any cross-agency transformations or quality checks
5. Writes the consolidated result to the **Unified Reporting Lakehouse**

### 3. Unified Reporting (Public Consumption)

The **Unified Reporting Lakehouse** resides in the DIF [PRD] Workspace and exposes the **`interagency_sharing` schema** containing the combined, multi-agency dataset. Read access is granted through a single **umbrella Entra group, `psde_url_interagencysharing_reader`**, which is assigned a **OneLake security role (`Grant` / `Read`) scoped to the `interagency_sharing` schema**.

Crucially, that umbrella group holds **no individual users**. Instead, each agency's existing **steward and developer groups** — `psde_ca-{agency}_data_stewards` and `psde_ca-{agency}_developers` — are **nested into it** (OneLake security resolves nested Entra group membership transitively). Every participant therefore gets visibility into the aggregated data without exposing any agency's raw operational tables.

This mirrors the inbound side: just as the DIF Workspace Identity is granted once via its Entra group, consumption is granted once to the umbrella group. **Onboarding a new agency's readers is purely a group-membership change — add that agency's two security groups to `psde_url_interagencysharing_reader`.** No new OneLake role, schema grant, or share is required; the single role scoped to `interagency_sharing` already covers everyone nested under the umbrella group.

### Implementation Notes (key constraints)

These are the non-obvious facts that make or break the implementation:

- **Cross-workspace MLVs cannot be read by four-part name.** `SELECT * FROM `Workspace`.`Lakehouse`.`url`.`vw_x`` fails with `TABLE_OR_VIEW_NOT_FOUND` across a workspace boundary — the catalog name lookup for a view/MLV object does not resolve cross-workspace, **even with full permissions** (Contributor was tested and still failed). The collector instead reads the materialized Delta by **OneLake path**:
  `abfss://{workspace_id}@onelake.dfs.fabric.microsoft.com/{lakehouse_id}/Tables/{source_schema}/{view}`
  Path-based reads hit OneLake storage directly and are governed by the scoped OneLake role, preserving least privilege.
- **GUIDs are resolved per environment, not hardcoded.** `NB_DIF_LAKEHOUSE_DEFINITIONS` resolves each agency's `workspace_id` / `lakehouse_id` from its display name via the Fabric REST API (under the admin/deploy identity) and stores them in `config.url_workspaces`. The collector reads those GUIDs from config and never calls the Fabric APIs — the least-privilege SP cannot list workspaces, so it must not need to.
- **Run order:** deploy and run DEFINITIONS first in each environment (populates GUIDs), then run Collect.
- **Refresh ownership:** the MLV refresh runs under the agency's identity and is the only thing that reads base tables; it stays entirely within the agency workspace and never widens the consumer's access.

### Benefits

| Concern | How This Architecture Addresses It |
|---------|-----------------------------------|
| **Least-privilege access** | Agencies expose only materialized views (`url` schema); a OneLake role scoped to that schema denies the DIF SP all base tables (verified `403`) |
| **Storage-enforced security** | OneLake security governs reads at the storage layer, independent of workspace role — consistent across Spark, SQL endpoint, and other engines |
| **No live base-table dependency** | MLVs persist a Delta snapshot, so consumers never resolve base tables at read time; only the agency-owned refresh job touches them |
| **Single audit trail** | One pipeline run = one lineage record for all agency data |
| **Schema governance** | If an agency's views break, only the DIF pipeline fails — not every consumer |
| **Scalable onboarding** | Adding a new agency = create their lakehouse + `url` MLVs + add their workspace to config + one Read share and OneLake role grant to the DIF group (inbound); nest their steward/developer groups into `psde_url_interagencysharing_reader` (outbound) — no new roles either direction |
| **Linear access scaling** | Each agency grants access once to DIF (hub-and-spoke), so grants grow with the number of agencies, not with the number of agency pairs |
| **Data deconfliction** | Agency-prefixed keys prevent PK/FK collisions after UNION |
