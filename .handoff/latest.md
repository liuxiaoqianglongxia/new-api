# ChatGPT / OpenCode Handoff Latest

Updated: 2026-06-16T11:30:00+08:00
Scope: project
Project: new-api
Directory: ~/projects/new-api
Mode: always-on-tail

## User goal

改进 AGENTS.md 并评估项目当前状态。

## What just happened

### 1. 改进 AGENTS.md
- 添加测试命令细化（单个包/函数测试）
- 添加前端质量门禁（typecheck, lint）
- 添加 Makefile 常用目标（dev, dev-api, dev-web, reset-setup）
- 添加迁移模式说明（GORM 自动迁移）
- 新增 Rule 9：Provider 适配器实现步骤
- 添加验证清单（7 项检查）
- 明确 web/default 是活跃前端，web/classic 是遗留版本
- 添加常见陷阱（7 个代理易犯错误）

### 2. 项目状态评估
- 后端：585 文件，111k 行 Go 代码，43 个测试文件（10.6% 覆盖率）
- 前端：909 个 TSX/TS 文件（React 19 + TypeScript）
- 20 个 AI 提供商适配器
- 开发活跃度低（最近 30 天仅 1 个 commit）
- ⚠️ 开发环境不完整（缺少 Go 和 Bun 工具链）

## Current state

- AGENTS.md 已改进（从 241 行增至 301 行）
- 项目代码完整但开发环境需配置
- 有未提交的修改（.env.example, .gitignore, AGENTS.md, docker-compose.dev.yml）
- 有未跟踪文件（.handoff/, docs/migration/, docs/ops/）

## Active files / paths

- ~/projects/new-api/AGENTS.md — 项目规则（已改进）
- ~/projects/new-api/.handoff/latest.md — 当前 handoff
- ~/projects/new-api/.handoff/status.md — 当前状态
- ~/projects/new-api/.handoff/report.md — 深度报告
- ~/projects/new-api/.handoff/archive/ — 归档

## Next likely action

1. 配置完整开发环境（安装 Go 1.22+ 和 Bun）
2. 运行 `go test ./...` 验证后端健康
3. 运行 `cd web/default && bun install && bun run typecheck && bun run lint`
4. 清理工作区（决定哪些修改需要提交）
5. 考虑增加测试覆盖率

## Constraints

- 不得修改 Windows 生产环境
- 不得输出 secrets
- 不得删除 .ai/
- Push/PR/release 需确认

## Secret status

No secrets included. Sensitive values redacted as <redacted>.
