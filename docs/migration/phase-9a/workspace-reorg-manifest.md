# Workspace Reorganization Manifest

**Date**: 2026-06-15
**Phase**: 9a - Workspace Cleanup

---

## Changes

### 1. Moved: one-api-reference → ~/references/

| From | To |
|------|-----|
| `~/projects/one-api-reference` | `~/references/one-api-reference` |

**Reason**: Reference repository, not an active project. Keeps `~/projects/` clean for working projects only.

### 2. Moved: migration-docs → docs/migration/phase-9a/

| From | To |
|------|-----|
| `~/projects/new-api-migration-docs/windows-current-state.md` | `docs/migration/phase-9a/windows-current-state.md` |

**Reason**: Migration docs belong inside the project they document. The standalone `new-api-migration-docs` directory has been removed.

### 3. Deleted: temp-new-api/

| Path | Contents |
|------|----------|
| `~/projects/new-api/temp-new-api/` | Upstream docs (channel settings, images, OpenAPI specs, translation glossaries) |

**Reason**: Residual copy of upstream documentation. Already available via `~/references/one-api-reference/docs/`.

### 4. Updated: .gitignore

Added `.ai/handoff/` to gitignore. Existing entries already cover `.env`, `data/`, `logs/`.

---

## Final Directory Structure

```
~/projects/
├── new-api/                    # Main project (Go source, git repo)
│   ├── docs/
│   │   ├── ops/                # Operations docs
│   │   │   ├── env-vars.md
│   │   │   ├── runbook.md
│   │   │   ├── windows-current-state.md
│   │   │   ├── wsl-dev-setup.md
│   │   │   └── wsl-server-1panel-deploy-plan.md
│   │   └── migration/
│   │       └── phase-9a/       # Migration documentation
│   │           ├── windows-current-state.md
│   │           └── workspace-reorg-manifest.md
│   ├── AGENTS.md
│   ├── CLAUDE.md
│   └── ...
└── (no more new-api-migration-docs or one-api-reference)

~/references/
└── one-api-reference/          # Upstream reference repo (read-only)
```

---

## Rollback

To undo these changes:

```bash
# Restore one-api-reference
mv ~/references/one-api-reference ~/projects/one-api-reference

# Restore migration docs
mkdir -p ~/projects/new-api-migration-docs
mv ~/projects/new-api/docs/migration/phase-9a/windows-current-state.md ~/projects/new-api-migration-docs/

# temp-new-api cannot be restored (was upstream docs, available via ~/references/one-api-reference)
```
