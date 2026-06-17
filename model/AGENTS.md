# model — 数据模型与数据库层

GORM 数据模型、数据库初始化、自动迁移、跨数据库兼容。

## OVERVIEW

所有数据表通过 GORM struct 定义，`InitDB()` 启动时自动迁移。必须同时兼容 SQLite + MySQL + PostgreSQL。

## STRUCTURE

```
model/
├── main.go              # InitDB(), migrateDB(), 跨DB变量 (commonGroupCol等)
├── user.go              # 用户模型
├── channel.go           # 通道模型
├── token.go             # API Key / Token 模型
├── log.go               # 请求日志模型
├── option.go            # 系统配置 (key-value)
├── ability.go           # 模型能力定义
├── channel_cache.go     # 通道缓存
├── billing*.go          # 计费相关模型
├── subscription*.go     # 订阅模型
├── payment*.go          # 支付模型
├── passkey.go           # WebAuthn/Passkey
├── midjourney.go        # Midjourney 任务
├── perf_metric.go       # 性能指标
└── *_test.go            # 模型层测试
```

## WHERE TO LOOK

| 任务 | 文件 | 说明 |
|------|------|------|
| 添加新表 | 新建 `<name>.go` + `main.go` AutoMigrate 列表 | struct tag 定义 DDL |
| 修改现有表 | struct 添加字段 + tag | GORM AutoMigrate 自动处理 ALTER |
| 跨 DB 兼容 | `main.go` → `commonGroupCol`, `commonKeyCol` 等 | PostgreSQL 用双引号, MySQL/SQLite 用反引号 |
| 原始 SQL | `main.go` 迁移函数 | 必须用 `commonGroupCol`/`commonKeyCol` 处理保留字 |
| 数据库初始化 | `main.go` → `InitDB()` | 连接 + 迁移 + 创建 root 用户 |
| 模型关联 | 各 model 文件 | 使用 GORM `ForeignKey`, `BelongsTo` 等 tag |

## 跨数据库兼容 (Rule 2)

```go
// main.go — 启动时根据数据库类型设置
var commonGroupCol string   // PostgreSQL: "group" / MySQL/SQLite: `group`
var commonKeyCol string     // PostgreSQL: "key"   / MySQL/SQLite: `key`
var commonTrueVal string    // PostgreSQL: true    / MySQL/SQLite: 1
var commonFalseVal string   // PostgreSQL: false   / MySQL/SQLite: 0
```

**原始 SQL 中使用**：
```sql
-- 正确 (跨DB兼容)
SELECT * FROM channels WHERE ` + commonGroupCol + ` = 'default'

-- 错误 (仅 MySQL)
SELECT * FROM channels WHERE `group` = 'default'
```

**禁止的 SQL 特性** (无跨 DB 回退方案时)：
- `GROUP_CONCAT`
- JSONB 运算符 (`->`, `->>`)
- `ALTER COLUMN` on SQLite

**JSON 存储**：使用 `TEXT` 类型而非 `JSONB`，应用层用 `common.Marshal`/`common.Unmarshal`。

## GORM AutoMigrate

`migrateDB()` 在 `main.go` 第 258 行，所有新模型必须添加到 `DB.AutoMigrate()` 调用中。

migrateDB 有两个版本：
- `migrateDB()` — 慢速完整迁移
- `migrateDBFast()` — 快速迁移 (跳过部分表)

```go
err := DB.AutoMigrate(
    &User{},
    &Channel{},
    &Token{},
    // ... 添加新模型到这里
)
```

## CONVENTIONS

- 模型文件命名：`<entity>.go` (小写单数)
- GORM tag 放在 struct 字段上：`gorm:"column:xxx;type:text;index"`
- JSON tag 使用 snake_case：`json:"user_name"`
- 时间字段使用 `int64` (Unix timestamp) 或 `time.Time`
- 关联查询优先使用 GORM `Preload()`，避免 N+1
- 软删除使用 GORM `DeletedAt` 字段

## ANTI-PATTERNS

- **不要** 用 `encoding/json` 直接序列化 — 使用 `common.Marshal`/`common.Unmarshal`
- **不要** 在迁移函数中写死列名引号 — 用 `commonGroupCol`/`commonKeyCol`
- **不要** 使用 `JSONB` 类型 — PostgreSQL 专用，用 `TEXT` 代替
- **不要** 直接 `ALTER COLUMN` — SQLite 不支持
- **不要** 在生产环境手动执行 SQL DDL — 依赖 GORM AutoMigrate
- **不要** 在 model 层添加业务逻辑 — model 只负责数据定义和简单 CRUD
