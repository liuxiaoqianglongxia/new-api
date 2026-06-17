# Version Audit - new-api

## Identity
- WSL: wsl-opencode
- Project root: /home/opencode/projects/new-api
- Project display name: new-api (AI API Gateway)
- Inferred canonical project: QuantumNous/new-api (fork of songquanpeng/one-api)
- Confidence: high
- Role guess: later-dev (active development fork with custom modifications)
- Evidence for role guess:
  - Git remote: https://github.com/Calcium-Ion/new-api.git
  - go.mod: module github.com/QuantumNous/new-api
  - Has local customizations (AGENTS.md, .env, docker-compose.override.yml, docs/)
  - Upstream is active (latest commit: aeea3fa)
  - Local dirty state includes AGENTS.md improvements and dev environment config

## Git
- Is git repo: YES (full clone with remote)
- Remote: origin → https://github.com/Calcium-Ion/new-api.git
- Branch: main (tracking origin/main)
- HEAD: aeea3fa "fix(keys): refresh API key form options (#5512)"
- Uncommitted:
  - Modified: .env.example (+260/-190 lines), .gitignore (+3), AGENTS.md (+223/-190), docker-compose.dev.yml (port change)
  - Untracked: .handoff/, docker-compose.override.yml, docs/migration/, docs/ops/, model/AGENTS.md, relay/channel/AGENTS.md
- Tags: none
- Submodules: none

## Tech Stack
- Languages: Go (backend), TypeScript/React (frontend)
- Frameworks: Gin (Go HTTP), React 19 + Base UI + Tailwind (frontend)
- Package managers: Go modules (backend), Bun (frontend)
- Runtime: Go 1.25.1, Node.js (for frontend dev)
- Database: PostgreSQL / MySQL / SQLite (multi-DB via GORM)
- Frontend: React 19 (web/default/ active, web/classic/ legacy)
- Backend: Go with Gin framework, 37+ AI provider adapters
- Deployment: Docker, docker-compose, Makefile, systemd

## Structure
- Important directories:
  - router/ — HTTP routing
  - controller/ — Request handlers
  - service/ — Business logic
  - model/ — Data models + GORM
  - relay/ — AI API relay/proxy (37 provider adapters in relay/channel/)
  - middleware/ — Auth, rate limiting, CORS
  - dto/ — Request/response structs
  - constant/ — Channel types, API types
  - types/ — Relay formats, errors
  - common/ — Shared utilities
  - web/default/ — Active frontend (React 19)
  - web/classic/ — Legacy frontend (DO NOT MODIFY)
  - docs/ — Documentation including migration/, ops/
  - .handoff/ — Project handoff state
  - .ai/ — AI configuration
  - .agents/ — Agent configuration
- Important entrypoints: main.go, router/, Dockerfile
- Config files: .env, .env.example, docker-compose.yml, docker-compose.dev.yml, docker-compose.override.yml, Makefile
- Build outputs: new-api binary
- Generated/cache directories: web/default/node_modules, web/default/dist
- Test directories: 43+ test files across packages (10.6% coverage)

## Runtime / Deployment Clues
- Ports: 3000 (default), 13001 (docker-compose.dev.yml), 13002 (docker-compose.override.yml — avoids conflict with 13001)
- Docker: calciumion/new-api:latest, Dockerfile (multi-stage: bun build + Go build)
- compose services: new-api, postgres, redis (optionally mysql)
- systemd: new-api.service
- scripts: Makefile (dev, dev-api, dev-web, reset-setup)
- env files present (names only): .env, .env.example
- local server evidence: configured for port 13001/13002 in WSL

## Similarity Fingerprints
- package/app names: new-api (QuantumNous), one-api (songquanpeng)
- service names: new-api
- key route names: /api/status, /v1/models, /v1/chat/completions, /v1/messages
- database schema: GORM auto-migration (tokens, channels, users, logs, quota_data, abilities, options)
- lockfile clues: go.mod (github.com/QuantumNous/new-api), bun.lock (web/)
- unique identifiers: SQL_DSN, REDIS_CONN_STRING, SYNC_FREQUENCY, SESSION_SECRET, NODE_NAME
- candidate duplicate names: references/one-api-reference (upstream parent project)

## Unique Value Candidates
- code: Custom AGENTS.md with project-specific rules (301 lines, improved)
- code: model/AGENTS.md, relay/channel/AGENTS.md (subdirectory agent rules)
- code: docker-compose.override.yml (WSL-specific port override)
- docs: docs/migration/phase-9a/ (migration documentation)
- docs: docs/ops/ (7 operational documents: env-vars, runbook, WSL dev setup, Docker engine install, 1Panel deploy, Windows state, phase-9b plan)
- configs: .env (production-like configuration with 24+ settings)
- configs: .env.example (customized with 260+ lines of documentation)

## Risk Notes
- secrets present (by filename only): .env (contains SQL_DSN, REDIS_CONN_STRING, SESSION_SECRET — all redacted)
- database/data dirs: data/ (SQLite fallback), PostgreSQL/Redis (via compose)
- production/server sensitive dirs: none specific
- large binary/data dirs: web/ (frontend assets, node_modules)
- deletion risk: MEDIUM — has uncommitted customizations that would be lost
- unknowns: Go/Bun toolchain not installed in this WSL (per .handoff/latest.md)

## Recommendation for D
- Recommendation: **keep** — active development project with custom documentation
- Exact reasons:
  - Fork with custom AGENTS.md improvements
  - 7 operational documents in docs/ops/ unique to this installation
  - docker-compose.override.yml for WSL-specific configuration
  - Upstream remote allows continued sync
  - .env contains operational configuration (not easily recoverable)
- Files D should inspect first:
  1. AGENTS.md (custom project rules)
  2. docs/ops/ (7 operational documents)
  3. .env (configuration)
  4. docker-compose.override.yml
  5. .handoff/latest.md (current state)
