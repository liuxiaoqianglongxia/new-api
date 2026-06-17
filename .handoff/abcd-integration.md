# ABCD Integration - new-api

## Cross-WSL Status

| WSL | Path | Status |
|-----|------|--------|
| A/B/C | ❌ | new-api exists only in D |
| D | ~/projects/new-api | Active fork with custom documentation |

## D Current State

- **Remote**: github.com/Calcium-Ion/new-api (origin)
- **Branch**: main, HEAD: aeea3fa
- **Uncommitted**: .env.example (+260/-190), .gitignore (+3), AGENTS.md (+223/-190), docker-compose.dev.yml
- **Untracked**: .handoff/, docker-compose.override.yml, docs/migration/, docs/ops/, model/AGENTS.md, relay/channel/AGENTS.md
- **Related**: ~/references/one-api-reference (upstream parent, clean clone, delete candidate)

## Integration Decisions

1. **Keep as-is** — D-only project, no cross-WSL conflicts
2. **Commit customizations** — AGENTS.md improvements and docs/ops/ should be committed
3. **Delete one-api-reference** — clean upstream clone, re-clonable

## Secrets (filenames only)

- .env (SQL_DSN, REDIS_CONN_STRING, SESSION_SECRET — all redacted in report)
