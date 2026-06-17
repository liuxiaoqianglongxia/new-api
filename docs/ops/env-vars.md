# 环境变量说明

**生成时间**: 2026-06-15  
**项目**: New API

---

## 一、核心配置

| 变量 | 用途 | 示例值 | 必填 |
|------|------|--------|------|
| PORT | 服务端口 | 3000 | 否 (默认 3000) |
| SQL_DSN | 数据库连接字符串 | postgresql://user:pass@host:5432/db | 是 |
| REDIS_CONN_STRING | Redis 连接字符串 | redis://:pass@host:6379 | 否 |
| SESSION_SECRET | 会话密钥（多节点必须） | random_string | 多节点必填 |

---

## 二、数据库配置

| 变量 | 用途 | 示例值 |
|------|------|--------|
| SQL_DSN | 主数据库连接 | postgresql://postgres:pass@postgres:5432/newapi |
| LOG_SQL_DSN | 日志数据库连接 | postgresql://postgres:pass@postgres:5432/logdb |
| SQLITE_PATH | SQLite 数据库路径 | /data/new-api.db |
| SQL_MAX_IDLE_CONNS | 最大空闲连接数 | 100 |
| SQL_MAX_OPEN_CONNS | 最大打开连接数 | 1000 |
| SQL_MAX_LIFETIME | 连接最大生命周期（秒） | 60 |

---

## 三、缓存配置

| 变量 | 用途 | 示例值 |
|------|------|--------|
| REDIS_CONN_STRING | Redis 连接 | redis://:password@redis:6379 |
| SYNC_FREQUENCY | 同步频率（秒） | 60 |
| MEMORY_CACHE_ENABLED | 启用内存缓存 | true |
| CHANNEL_UPDATE_FREQUENCY | 渠道更新频率（秒） | 30 |
| BATCH_UPDATE_ENABLED | 启用批量更新 | true |
| BATCH_UPDATE_INTERVAL | 批量更新间隔（秒） | 5 |

---

## 四、性能配置

| 变量 | 用途 | 示例值 |
|------|------|--------|
| GIN_MODE | Gin 框架模式 | release |
| MEMORY_CACHE_ENABLED | 内存缓存 | true |
| SQL_MAX_IDLE_CONNS | 数据库空闲连接 | 20 |
| SQL_MAX_OPEN_CONNS | 数据库打开连接 | 100 |
| SQL_CONN_MAX_LIFETIME | 连接生命周期 | 60 |
| BatchUpdateInterval | 批量更新间隔 | 300 |
| GlobalApiRateLimit | 全局 API 速率限制 | 0 (无限制) |

---

## 五、日志配置

| 变量 | 用途 | 示例值 |
|------|------|--------|
| LOG_DIR | 日志目录 | /app/logs |
| ERROR_LOG_ENABLED | 启用错误日志 | true |

---

## 六、代理配置

| 变量 | 用途 | 示例值 |
|------|------|--------|
| HTTPS_PROXY | HTTPS 代理 | http://127.0.0.1:7897 |
| HTTP_PROXY | HTTP 代理 | http://127.0.0.1:7897 |

---

## 七、调试配置

| 变量 | 用途 | 示例值 |
|------|------|--------|
| DEBUG | 启用调试模式 | true |
| ENABLE_PPROF | 启用 pprof | true |

---

## 八、安全配置

| 变量 | 用途 | 示例值 |
|------|------|--------|
| SESSION_SECRET | 会话密钥 | random_string |
| TLS_INSECURE_SKIP_VERIFY | 跳过 TLS 验证 | false |

---

## 九、功能配置

| 变量 | 用途 | 示例值 |
|------|------|--------|
| UPDATE_TASK | 启用更新任务 | true |
| GENERATE_DEFAULT_TOKEN | 生成默认 token | false |
| GET_MEDIA_TOKEN | 统计图片 token | true |
| GET_MEDIA_TOKEN_NOT_STREAM | 非流模式统计图片 | false |

---

## 十、超时配置

| 变量 | 用途 | 示例值 |
|------|------|--------|
| RELAY_TIMEOUT | 请求超时（秒） | 0 (不限制) |
| RELAY_IDLE_CONN_TIMEOUT | 空闲连接超时（秒） | 90 |
| STREAMING_TIMEOUT | 流模式超时（秒） | 300 |

---

## 十一、节点配置

| 变量 | 用途 | 示例值 |
|------|------|--------|
| NODE_NAME | 节点名称 | new-api-node-1 |
| NODE_TYPE | 节点类型 | master |

---

## 十二、敏感变量

**以下变量包含敏感信息，不要提交到 git**:

- SQL_DSN (包含数据库密码)
- REDIS_CONN_STRING (包含 Redis 密码)
- SESSION_SECRET (会话密钥)
- HTTPS_PROXY / HTTP_PROXY (代理地址)

**使用 .env 文件管理，并添加到 .gitignore**

---

**文档生成完成**
