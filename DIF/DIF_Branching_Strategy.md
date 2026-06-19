# DIF Branching Strategy & Deployment Pipeline

## Overview

The PSDE Data Ingestion Framework (DIF) uses a **Git-based, branch-per-environment** deployment strategy. Each Fabric workspace is connected to a dedicated Git branch via Fabric's native Git integration. Code promotion flows through branches using Pull Requests (PRs) in Azure DevOps, ensuring every change is reviewed, tracked, and auditable.

This approach follows Microsoft's **Option 1 — Git-Based Deployments** pattern, where Git serves as the single source of truth and all deployments originate from the repository.

## Branch-to-Workspace Mapping

| Branch | Fabric Workspace | Purpose |
|--------|-----------------|---------|
| `dev` | **DIF [DEV]** | Active development workspace. Developers merge feature branches here via PRs. |
| `main` | **DIF [PRD]** | Production workspace. Receives validated content from `dev` via PR. |

**Repository:** `https://dev.azure.com/wfca-com/PSDE/_git/psde_dif`

## How It Works

```
Feature branches ──PR──► dev ──PR──► main
                          │            │
                     DIF [DEV]    DIF [PRD]
```

1. Each Fabric workspace is connected to its corresponding branch via **Workspace Settings → Git integration** in the Fabric portal.
2. When a PR is merged into a branch, the connected Fabric workspace automatically syncs to reflect the updated branch content.
3. Promotion between environments is controlled entirely through Azure DevOps PRs, providing review gates and approval workflows at every stage.
4. Branch protection policies enforce that no one can push directly to `dev` or `main` — all changes must go through a PR.

## Developer Workflow

The recommended day-to-day workflow uses a **persistent private Fabric workspace** per developer. See [DIF Development Workflow](DIF_Development_Workflow.md) for the full operational pattern, including workspace setup, connection naming, and when to use branch-out workspaces.

### Initial Setup (One-Time)

1. **Set up your persistent private workspace** (`DIF_<developer>_DEV`) and connect it to the `psde_dif` repository via **Workspace Settings → Git integration** in the Fabric portal.
2. Complete one-time workspace identity and connection setup per the [DIF Development Workflow](DIF_Development_Workflow.md#one-time-setup-per-persistent-workspace) guide.
3. Optionally, clone the repo locally for branch management outside Fabric:
   ```bash
   git clone https://wfca-com@dev.azure.com/wfca-com/PSDE/_git/psde_dif
   cd psde_dif
   git fetch origin
   ```

### Day-to-Day Development

#### 1. Create a Feature Branch

In your persistent private workspace, use the **Source Control** panel to switch to a new feature branch based on `dev`. Branches can also be created locally:

```bash
git checkout dev
git pull origin dev
git checkout -b feature/<your-name>_<date>_<description>
git push -u origin feature/<your-branch-name>
```

**Naming convention:** `feature/<developer>_<MM.DD.YY>_<short-description>`
Example: `feature/Jerid_4.24.26_NewIngestionPipeline`

#### 2. Make and Commit Changes

- Add or modify notebooks, pipelines, environments, or other Fabric item definitions in your private workspace.
- Commit frequently with clear messages via the **Source Control** panel in Fabric or the git CLI:

```bash
git add .
git commit -m "Add ingestion pipeline for source X"
git push
```

#### 3. Create a Pull Request → `dev`

- In Azure DevOps, create a PR from your feature branch into `dev`.
- Add reviewers as required by the branch policy.
- Once approved and merged, the **DIF [DEV]** workspace automatically syncs with the updated `dev` branch.
- Validate your changes in the DEV workspace.

#### 4. Promote to Production

- Once development is validated in **DIF [DEV]**, create a PR from `dev` → `main` in Azure DevOps.
- This PR should have stricter review requirements (e.g., multiple approvers).
- After merge, the **DIF [PRD]** workspace syncs automatically.

### Hotfix Process

For urgent production fixes that cannot wait for the normal promotion cycle:

1. Create a hotfix branch from `main`:
   ```bash
   git checkout main
   git pull origin main
   git checkout -b hotfix/<description>
   ```
2. Apply the fix and push:
   ```bash
   git add .
   git commit -m "Hotfix: <description>"
   git push -u origin hotfix/<description>
   ```
3. Create a PR from the hotfix branch into `main`. After review and merge, PRD syncs.
4. **Important:** Merge the fix back into `dev` to keep branches aligned:
   ```bash
   git checkout dev
   git pull origin dev
   git merge main
   git push origin dev
   ```

## Branch Protection Policies

The following policies should be configured in Azure DevOps under **Repos → Branches → Branch policies**:

| Policy | `dev` | `main` |
|--------|-------|--------|
| Require PR (no direct push) | Yes | Yes |
| Minimum reviewers | 1 | 1–2 |
| Requestors can approve own PRs | Yes | No |
| Check for linked work items | Optional | Recommended |
| Comment resolution required | No | Yes |

## Important Considerations

- **Data is NOT tracked in Git.** Lakehouse tables, files, and data are workspace-specific. Only Fabric item definitions (notebooks, pipelines, environments, etc.) are version-controlled.
- **Environment publishing.** After syncing environments from Git, you must **publish** them in each workspace for changes to take effect.
- **Custom pool references** are workspace-scoped. If using custom Spark pools, update pool IDs after syncing to a different workspace.
- **Commit size limit.** Each commit is limited to 150 MB. Avoid committing large binary files or custom libraries that exceed this limit.
- **Workspace sync.** After a PR merge, Fabric may prompt users in the workspace to accept the update. Click **Update all** in the Source Control panel to sync.

## References

- [Manage Branches in Microsoft Fabric Workspaces](https://learn.microsoft.com/fabric/cicd/git-integration/manage-branches)
- [Choose the Best Fabric CI/CD Workflow Option](https://learn.microsoft.com/fabric/cicd/manage-deployment)
- [Best Practices for Lifecycle Management in Fabric](https://learn.microsoft.com/fabric/cicd/best-practices-cicd)
- [Get Started with Git Integration](https://learn.microsoft.com/fabric/cicd/git-integration/git-get-started)
