# 当前项目深度验收报告 (latest)

> 📋 **这是当前项目深度验收报告 latest**
> 日常 handoff 请看 `.ai/handoff/latest.md`
> 深度验收时，复制本文件 + handoff latest 给 ChatGPT

**最后更新**: 2026-06-15
**当前阶段**: Phase 9X Artifact Architecture Review

---

## 一、最新验收报告

# Phase 9X Artifact Architecture Review — 最终报告

> **Phase**: 9X Artifact Architecture Review
> **时间**: 2026-06-15T21:10:00+08:00
> **状态**: ✅ COMPLETE
> **Coordinator**: qwen3.7-plus (Sisyphus)

---

## 一、旧报告体系的问题

### 1.1 核心痛点

| 问题 | 影响 |
|------|------|
| 深度报告只在全局 `~/.config/opencode/reports/acceptance/` | 用户找 New API 资料时要在两个目录间跳转 |
| handoff 在项目内，report 在全局 | 两个入口割裂，用户不知道该看哪个 |
| 全局 acceptance 是"主入口" | 多项目时全局目录变成大杂烩 |
| `.ai/` 只有 handoff 子目录 | reports、raw、status 无处安放 |
| `.gitignore` 只忽略 `.ai/handoff/` | `.ai/reports/` 等子目录可能被 git 跟踪 |
| 没有全局 `.ai/` 标准 | 每个项目各自为政，无法复用 |

### 1.2 用户核心需求

1. 日常复制给 ChatGPT，只复制 **1 个文件**：`.ai/handoff/latest.md`
2. 深度验收时，最多再复制 **1 个文件**：`.ai/reports/latest.md`
3. 人工使用时最多两个文件
4. 其他索引、原始文件、历史归档由 OpenCode 设计和维护
5. 全局目录只做跨项目索引，不再作为人工主入口

---

## 二、新项目内 `.ai/` 目录标准（全局通用）

```
~/projects/<project>/.ai/
├── README.md              # 项目 AI 工作区说明
├── handoff/
│   ├── latest.md          # 日常 handoff（给 ChatGPT 的唯一文件）
│   ├── live-status.md     # 任务运行中状态
│   ├── INDEX.md           # 历史 handoff 索引
│   └── YYYY-MM-DD/        # 按日期归档
├── reports/
│   ├── latest.md          # 深度验收报告（验收时额外给 ChatGPT）
│   ├── INDEX.md           # 历史报告索引
│   └── YYYY-MM-DD/        # 按日期归档
├── raw/
│   └── YYYY-MM-DD/        # 原始证据/研究材料
└── status/
    └── current.md         # 自动化脚本读取
```

### 设计原则

1. `handoff/latest.md` 是日常发给 ChatGPT 的唯一文件
2. `reports/latest.md` 是深度验收时额外发给 ChatGPT 的唯一文件
3. 历史归档进入日期目录
4. 原始研究和命令摘要进入 raw
5. 全局 acceptance 只保留索引和指针
6. 全局 handoff 只保留跨项目索引
7. 项目报告正文必须在项目内
8. `.ai/` 默认不进入 git
9. 每个任务最终回复必须打印 5 个路径
10. 不允许 latest.md 缩水成几行状态摘要（handoff ≥ 80 行，report ≥ 50 行）

---

## 三、专家组成员和任务

| 角色 | 模型 | 职责 | 状态 |
|------|------|------|------|
| Architect/Coordinator | qwen3.7-plus | 提出最终结构、裁决、落地、验收 | ✅ |
| Harsh Reviewer | MiniMax-M3 (综合模拟) | 专门挑刺 | ✅ |
| Read-only Auditor | MiniMax-M3 (实际审计) | 只读检查当前真实文件 | ✅ |
| Workflow Advisor | MiniMax-M3 (综合模拟) | 长期使用方案建议 | ✅ |

---

## 四、毒舌 Reviewer 的主要批评

### 4.1 路径是否过多

> "用户说只要 2 个文件，但你搞出了 README.md、INDEX.md、live-status.md、current.md、raw/、resume/ 共 20 个文件。虽然用户日常确实只看 2 个，但目录结构本身有 7 个子目录/文件类型，新人第一次看 `.ai/` 会懵。"

**Coordinator 裁决**: README.md 第一行就写清"用户只需要记住两个文件"，其他文件标注"❌ 日常不需要看"。可接受。

### 4.2 `latest.md` 和 `reports/latest.md` 是否容易混淆

> "两个 `latest.md` 在不同目录下，一个叫 handoff latest，一个叫 report latest。用户复制时可能搞混。"

**Coordinator 裁决**: 路径本身已区分（`handoff/latest.md` vs `reports/latest.md`），README.md 明确标注用途。可接受，但任务结束回复必须固定打印两个完整路径。

### 4.3 `raw/` 是否会变成垃圾堆

> "raw/ 没有清理机制，长期积累会变成无人问津的垃圾目录。"

**Coordinator 裁决**: raw/ 是 OpenCode 内部使用，用户不需要看。暂不加入自动清理，但 INDEX.md 不索引 raw/ 内容。

### 4.4 全局索引是否还有必要

> "既然全局目录不是日常入口，为什么还要保留 INDEX.md 和 LATEST.md？不如直接删掉。"

**Coordinator 裁决**: 全局索引保留作为跨项目导航。多项目场景下，用户可能需要一个"所有项目入口在哪"的总览。不删除，但明确标注"legacy/global copy"。

### 4.5 是否真的只需 2 个文件

> "说是 2 个文件，但 handoff latest 里引用了 live-status.md、README.md、INDEX.md。用户看到引用会不会也去复制？"

**Coordinator 裁决**: handoff latest 里的引用是"如需了解"，不是"必须复制"。README.md 明确标注 ✅/❌ 区分。可接受。

### 4.6 是否会污染 git

> "`.gitignore` 之前只忽略 `.ai/handoff/`，现在改成 `.ai/` 整目录。但如果有人手动 `git add -f` 呢？"

**Coordinator 裁决**: `.gitignore` 已改为 `.ai/` 覆盖全部子目录。手动 force-add 是用户主动行为，不在防范范围。

---

## 五、只读 Auditor 的发现

### 5.1 审计前状态（Phase 1）

| 项目 | 发现 |
|------|------|
| `.ai/` 文件数 | 12 个（全部在 handoff/ 和 resume/） |
| `.ai/reports/` | ❌ 不存在 |
| `.ai/README.md` | ❌ 不存在 |
| `.ai/raw/` | ❌ 不存在 |
| `.ai/status/` | ❌ 不存在 |
| `.gitignore` | ⚠️ 只忽略 `.ai/handoff/` |
| 全局 acceptance | 11 个文件（3 个 New API 相关深度报告） |
| 全局 handoff | 6 个文件 |
| handoff/latest.md | 指向全局 `~/.config/opencode/reports/acceptance/LATEST.md` |

### 5.2 审计后状态（Phase 7 验证）

| 项目 | 发现 | 状态 |
|------|------|------|
| `.ai/` 文件数 | 20 个 | ✅ +8 新增 |
| `.ai/reports/` | 存在，含 3 个归档报告 + latest + INDEX | ✅ |
| `.ai/README.md` | 存在，58 行 | ✅ |
| `.ai/raw/` | 存在（空） | ✅ |
| `.ai/status/current.md` | 存在，16 行 | ✅ |
| `.gitignore` | `.ai/` 覆盖全目录 | ✅ |
| reports/latest.md | 386 行（完整嵌入 Continuity Audit） | ✅ 非缩水 |
| handoff/latest.md | 158 行 | ✅ 非缩水 |
| 全局 INDEX.md | 指向项目内主入口 | ✅ |
| 全局 LATEST.md | 指向项目内主入口 | ✅ |
| AGENTS.md | 198 行，含 AI Artifact Policy | ✅ |

---

## 六、Workflow Advisor 的建议

### 6.1 新项目复用

1. 使用全局模板：`cp -r ~/.config/opencode/templates/project-ai/* <project>/.ai/`
2. 替换占位符：`<PROJECT_NAME>`, `<DAILY_HANDOFF_PATH>`, `<DEEP_REPORT_PATH>`
3. 在 `.gitignore` 添加 `.ai/`
4. 在 `project-registry.md` 登记时勾选"是否有 .ai/ 结构"

### 6.2 旧项目迁移

1. 按 `legacy-project-intake-playbook.md` 阶段 4 执行
2. 创建 `.ai/` 目录结构
3. 从全局 acceptance 复制相关报告到项目内
4. 更新 `.gitignore`

### 6.3 ChatGPT 验收流程

1. 日常：复制 `.ai/handoff/latest.md`
2. 深度验收：额外复制 `.ai/reports/latest.md`
3. 最多 2 个文件，无需翻全局目录

### 6.4 OpenCode 最终回复应打印

```
📁 Daily handoff:    ~/projects/<project>/.ai/handoff/latest.md
📋 Deep report:      ~/projects/<project>/.ai/reports/latest.md
📊 Live status:      ~/projects/<project>/.ai/handoff/live-status.md
📖 Project AI:       ~/projects/<project>/.ai/README.md
🌐 Global index:     ~/.config/opencode/reports/acceptance/INDEX.md
```

---

## 七、Coordinator 最终采纳方案

### 7.1 采纳的设计决策

| 决策 | 理由 |
|------|------|
| 双入口（handoff + reports） | 日常和深度验收场景分离，用户按需选择 |
| `.ai/` 整目录 git-ignore | 简单、安全、无遗漏 |
| 全局目录保留但降级为导航 | 多项目场景需要跨项目总览 |
| 全局模板 7 个文件 | 新项目一键初始化 |
| AGENTS.md 只引用全局规则 | 避免重复，保持 < 220 行 |
| latest.md 行数下限 | 防止缩水（handoff ≥ 80，report ≥ 50） |

### 7.2 未采纳的建议

| 建议 | 理由 |
|------|------|
| 删除全局 acceptance 目录 | 历史兼容，多项目导航需要 |
| raw/ 自动清理 | 过早优化，暂不实施 |
| 合并 handoff/latest.md 和 reports/latest.md 为一个文件 | 职责不同，合并会过长 |

---

## 八、实际创建/更新的文件

### 8.1 项目内新增文件（8 个）

```
~/projects/new-api/.ai/README.md                                              (58 行)
~/projects/new-api/.ai/reports/latest.md                                      (386 行)
~/projects/new-api/.ai/reports/INDEX.md                                       (16 行)
~/projects/new-api/.ai/reports/2026-06-15/PHASE_9X_CONTINUITY_AUDIT_REPORT.md (361 行)
~/projects/new-api/.ai/reports/2026-06-15/PHASE_9X_ARTIFACT_HYGIENE_REPORT.md (71 行)
~/projects/new-api/.ai/reports/2026-06-15/PHASE_9X_PROJECT_GOVERNANCE_DOCKER_AND_NEW_API_WSL_DEV_REPORT.md (370 行)
~/projects/new-api/.ai/handoff/INDEX.md                                       (15 行)
~/projects/new-api/.ai/status/current.md                                      (16 行)
```

### 8.2 项目内更新文件（3 个）

```
~/projects/new-api/AGENTS.md          (188 → 198 行，+AI Artifact Policy)
~/projects/new-api/.gitignore         (.ai/handoff/ → .ai/)
~/projects/new-api/.ai/handoff/live-status.md  (更新为 Phase 9X Artifact Architecture Review)
```

### 8.3 全局规则更新（3 个）

```
~/.config/opencode/rules/chatgpt-handoff-policy.md           (+项目 .ai/ 标准)
~/.config/opencode/rules/finalization-watchdog-policy.md      (+六-b 项目内验证)
~/.config/opencode/rules/parallel-task-convergence-policy.md  (+十 项目 .ai/ 产物收敛)
```

### 8.4 全局参考更新（4 个）

```
~/.config/opencode/references/workspace-standards.md            (.ai/ 完整标准)
~/.config/opencode/references/project-registry.md               (+.ai/ 结构字段)
~/.config/opencode/references/agent-instructions-standards.md   (+AI Artifact Policy)
~/.config/opencode/references/legacy-project-intake-playbook.md (+.ai/ 创建步骤)
```

### 8.5 全局模板创建（7 个）

```
~/.config/opencode/templates/project-ai/README.md
~/.config/opencode/templates/project-ai/handoff-latest-template.md
~/.config/opencode/templates/project-ai/report-latest-template.md
~/.config/opencode/templates/project-ai/handoff-index-template.md
~/.config/opencode/templates/project-ai/report-index-template.md
~/.config/opencode/templates/project-ai/live-status-template.md
~/.config/opencode/templates/project-ai/raw-evidence-map-template.md
```

### 8.6 全局索引更新（3 个）

```
~/.config/opencode/reports/acceptance/INDEX.md     (重写为跨项目导航)
~/.config/opencode/reports/acceptance/LATEST.md    (重写为指向项目内)
~/.config/opencode/handoff/global/INDEX.md         (重写为跨项目导航)
```

---

## 九、全局化追加内容

### 9.1 哪些内容进入全局 rules

- `chatgpt-handoff-policy.md`: 12 条强制要求（项目 .ai/ 标准）
- `finalization-watchdog-policy.md`: 项目内 .ai/ 验证检查
- `parallel-task-convergence-policy.md`: 项目 .ai/ 产物收敛检查

### 9.2 哪些内容进入全局 references

- `workspace-standards.md`: 完整 `.ai/` 目录结构 + 8 条规则
- `project-registry.md`: 新增"是否有 .ai/ 结构"字段
- `agent-instructions-standards.md`: 第 7 核心区域 "AI Artifact Policy"
- `legacy-project-intake-playbook.md`: 阶段 4 新增 `.ai/` 创建步骤

### 9.3 哪些内容进入全局 templates

- 7 个模板文件，含占位符 `<PROJECT_NAME>`, `<PHASE>`, `<DATE>`, `<STATUS>`, `<DAILY_HANDOFF_PATH>`, `<DEEP_REPORT_PATH>`
- 不含任何 New API 专属内容
- 可用于任何新项目

### 9.4 哪些内容留在 New API 项目 AGENTS.md

- 仅 1 小节 "AI Artifact Policy"（6 行）
- 引用全局规则路径，不复制全文
- AGENTS.md 保持 198 行（< 220 行限制）

### 9.5 为什么深度报告必须项目内保存

1. 用户找项目资料时，理应在项目目录内找到一切
2. 全局目录是多项目共享的，项目报告混入会造成混乱
3. 项目内报告跟随项目生命周期（归档、迁移、删除）
4. 全局副本只作兼容和索引

### 9.6 为什么全局目录只做索引

1. 全局目录无法感知项目生命周期
2. 多项目时全局目录会膨胀
3. 用户日常只需项目内 2 个文件
4. 全局索引解决"我有哪些项目、入口在哪"的总览需求

### 9.7 以后新项目如何复用

```bash
# 1. 创建 .ai/ 目录
mkdir -p <project>/.ai/{handoff,reports,raw,status}

# 2. 复制模板
cp ~/.config/opencode/templates/project-ai/README.md <project>/.ai/
cp ~/.config/opencode/templates/project-ai/handoff-latest-template.md <project>/.ai/handoff/latest.md
cp ~/.config/opencode/templates/project-ai/report-latest-template.md <project>/.ai/reports/latest.md
# ... 其他模板

# 3. 替换占位符
sed -i 's/<PROJECT_NAME>/my-project/g' <project>/.ai/README.md
# ... 其他占位符

# 4. 更新 .gitignore
echo ".ai/" >> <project>/.gitignore

# 5. 在 project-registry.md 登记
```

### 9.8 以后旧项目迁移如何复用

1. 按 `legacy-project-intake-playbook.md` 阶段 4 执行
2. 创建 `.ai/` 目录结构（同上）
3. 从全局 acceptance 复制相关报告到项目内 `.ai/reports/YYYY-MM-DD/`
4. 生成 `reports/latest.md`（嵌入最新报告全文）
5. 更新 `.gitignore`
6. 在 `project-registry.md` 标注"是否有 .ai/ 结构 = 是"

---

## 十、用户以后默认复制哪个文件

**日常**：`~/projects/<project>/.ai/handoff/latest.md`

**深度验收**：额外复制 `~/projects/<project>/.ai/reports/latest.md`

**最多 2 个文件，无需翻全局目录。**

---

## 十一、全局索引如何定位

- 全局 acceptance INDEX: `~/.config/opencode/reports/acceptance/INDEX.md`
- 全局 acceptance LATEST: `~/.config/opencode/reports/acceptance/LATEST.md`
- 全局 handoff INDEX: `~/.config/opencode/handoff/global/INDEX.md`

全局索引已标注"⚠️ 全局目录不是用户日常入口"，指向项目内主入口。

---

## 十二、安全检查清单

| 检查项 | 结果 |
|--------|------|
| 是否删除历史文件 | ❌ 否 |
| 是否修改 Windows New API | ❌ 否 |
| 是否读取或输出敏感值 | ❌ 否 |
| 是否停止 Docker 容器 | ❌ 否 |
| 是否 commit/push/PR | ❌ 否 |
| 是否修改 OpenCode 核心配置 | ❌ 否 |
| `.ai/` 是否被 git 忽略 | ✅ 是 |
| 全局报告是否保留 | ✅ 是 |

---

## 十三、验证结果汇总

| 验证项 | 命令 | 结果 |
|--------|------|------|
| handoff latest 存在 | `test -s .ai/handoff/latest.md` | ✅ |
| report latest 存在 | `test -s .ai/reports/latest.md` | ✅ |
| README 存在 | `test -s .ai/README.md` | ✅ |
| reports INDEX 存在 | `test -s .ai/reports/INDEX.md` | ✅ |
| handoff INDEX 存在 | `test -s .ai/handoff/INDEX.md` | ✅ |
| status current 存在 | `test -s .ai/status/current.md` | ✅ |
| handoff 行数 ≥ 80 | `wc -l` = 158 | ✅ |
| report 行数 ≥ 50 | `wc -l` = 386 | ✅ |
| .ai/ 被 git 忽略 | `git check-ignore` | ✅ 全部匹配 |
| 全局索引指向项目内 | `grep` | ✅ |
| AGENTS.md < 220 行 | `wc -l` = 198 | ✅ |
| 模板文件 7 个 | `ls` = 7 | ✅ |
| 3 个规则已更新 | `grep` | ✅ |
| 4 个参考已更新 | `grep` | ✅ |

---

## 十四、是否可以进入 Phase 9C

✅ **可以进入 Phase 9C**。

前提条件：
1. 项目内 `.ai/` 报告体系已建立
2. 全局规范已更新
3. 全局模板已创建
4. handoff 和 report latest 均非缩水
5. 所有验证通过

Phase 9C 下一步：wsl-server discovery → 1Panel 画像 → 生产 Compose → 用户确认后部署。

---

**报告版本**: v1.0
**Coordinator**: qwen3.7-plus (Sisyphus)
**生成时间**: 2026-06-15T21:10:00+08:00

---

## 二、历史报告索引

See `.ai/reports/INDEX.md`

---

## 三、敏感内容检查

本文件不含任何 token/key/secret/password。
