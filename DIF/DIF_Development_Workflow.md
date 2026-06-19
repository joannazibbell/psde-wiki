# DIF Development Workflow (Efficient Pattern)

## Purpose

This page defines the recommended day-to-day development workflow for DIF in Microsoft Fabric with the least operational overhead while preserving governance, security, and PR-based change control.

## Executive Summary

Use a **persistent private developer workspace** per engineer and **switch Git branches** inside that same workspace for each feature.

Do **not** create a new workspace identity and new connections for every feature branch-out workspace unless strict runtime isolation is required.

## Why This Pattern

Branching out to a new workspace for every feature adds repeated setup work:

- Create workspace identity
- Grant identity access to external resources
- Recreate/rebind connections
- Reassign workspace roles

Microsoft Fabric lifecycle guidance supports a lighter model:

- Keep a private workspace and reuse it for future branches
- Switch the workspace Git connection to the next feature branch
- Commit and PR back to integration branches

## Target Operating Model

### Long-lived workspaces

| Workspace | Lifecycle | Workspace Identity (WI) | Notes |
|---|---|---|---|
| `DIF [DEV]` | Persistent | Required | Integration workspace for team validation |
| `DIF [PRD]` | Persistent | Required | Production workspace |
| `DIF_<developer>_DEV` | Persistent | Required | Private authoring/testing workspace per developer |

### Short-lived workspaces

| Workspace Type | Lifecycle | WI | Use Case |
|---|---|---|---|
| Feature branch-out workspace | Ephemeral | Optional (only if needed) | Use only when strict isolated runtime is needed |

## One-Time Setup Per Persistent Workspace

1. Create Workspace Identity in the workspace.
2. Grant the WI required access on target resources (ADLS, SQL, etc.).
3. Create workspace-scoped connections using **Authentication kind = Workspace identity**.
4. Use the connection naming convention below so branch switching does not require connection recreation.
5. Add Entra groups with least privilege:
   - `psde_dif_admins` -> `Admin`
   - `psde_dif_dataengineers` -> `Contributor`
6. For pipeline-to-pipeline invocation with WI, ensure the calling workspace's WI has at least **Contributor** role in the target workspace.

## Connection Naming Convention (Persistent Developer Workspaces)

Use a naming convention that preserves the current `CON_DIF_DEV_FABRIC_*` pattern and adds developer identity for private workspaces.

Pattern:

- `CON_DIF_DEV_<OWNER>_FABRIC_<PURPOSE>`

Where:

- `<OWNER>` = short uppercase user/team identifier (example: `JFORTNEY`, `NARPATH`)
- `<PURPOSE>` = connector purpose/type (example: `PIPELINES`, `NOTEBOOKS`, `SQL`)

Examples for a private workspace owned by JFORTNEY:

- `CON_DIF_DEV_JFORTNEY_FABRIC_PIPELINES`
- `CON_DIF_DEV_JFORTNEY_FABRIC_NOTEBOOKS`
- `CON_DIF_DEV_JFORTNEY_FABRIC_SQL`

Shared environment convention remains unchanged:

- `CON_DIF_DEV_FABRIC_PIPELINES`
- `CON_DIF_DEV_FABRIC_NOTEBOOKS`
- `CON_DIF_DEV_FABRIC_SQL`

Notes:

- Keep names uppercase with underscores for consistency and easy filtering.
- Do not embed branch names in connection names. Connections are workspace-scoped and reused across branch switches.
- Keep one stable connection set per persistent workspace and update secrets/permissions in place as needed.

## Day-to-Day Developer Workflow (Recommended)

1. Open your persistent private workspace (`DIF_<developer>_DEV`).
2. Source Control -> Switch branch to a new or existing feature branch.
3. Implement and test changes in the workspace.
4. Commit to the feature branch.
5. Create PR to `dev`.
6. After merge, sync `DIF [DEV]` and validate.
7. Promote via PRs (`dev` -> `main`) per branch strategy.

## When to Use Branch-Out Instead

Use branch-out to a separate workspace only if one of these is true:

- You need hard runtime isolation for high-risk testing.
- You must run parallel experiments that could interfere in a shared/private workspace.
- You need temporary environment-specific dependencies that should not live in your persistent workspace.

If branching out to a new workspace and you need runtime execution there, complete WI + connection setup for that workspace.

## Connection and Deployment Considerations

- Connections are workspace-scoped, not Git-scoped.
- Item definitions move through Git/PRs; connection objects do not — Fabric's Git integration does not track connections by platform design.
- Keep environment-specific connection mapping in deployment process (deployment rules/parameterization where applicable).
- Avoid hardcoding name-based or invalid GUID references in pipeline invoke activities.

## Practical Guardrails

- Keep private developer workspaces long-lived and reusable.
- Keep feature branches short-lived.
- Use small commits and PRs.
- Validate in `DIF [DEV]` before promotion.
- Reserve production deployment rights to a limited approver set.

## Related Pages

- [DIF Branching Strategy & Deployment Pipeline](DIF_Branching_Strategy.md)
- [Fabric Tenant Settings Review](Fabric_Tenant_Settings_Review.md)

## References

- https://learn.microsoft.com/fabric/cicd/best-practices-cicd
- https://learn.microsoft.com/fabric/cicd/git-integration/manage-branches
- https://learn.microsoft.com/fabric/cicd/git-integration/branched-workspace
- https://learn.microsoft.com/fabric/data-factory/invoke-pipeline-activity
- https://learn.microsoft.com/fabric/security/workspace-identity-authenticate
