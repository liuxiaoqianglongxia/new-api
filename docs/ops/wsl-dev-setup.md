# WSL 开发环境设置指南

**生成时间**: 2026-06-15  
**项目**: New API  
**环境**: WSL OpenCode

---

## 一、环境要求

- WSL2 (Ubuntu 24.04+)
- Go 1.22+
- Node.js 22+
- Bun (前端包管理)
- Docker & Docker Compose
- Redis (可选，用于缓存)
- SQLite/MySQL/PostgreSQL (数据库)

---

## 二、端口规划

| 服务 | 端口 | 说明 |
|------|------|------|
| New API Web | 13002 | WSL 开发环境（实际端口；原规划 13001 被 WSL 系统级 socket 永久占用，无法释放） |
| Redis | 16379 | WSL 开发环境 |
| PostgreSQL | 15432 | WSL 开发环境 |
| MySQL | 13306 | WSL 开发环境（备选） |

**注意**: 
- Windows New API 使用 3001 端口
- OpenChamber 使用 3002 端口
- Paseo 使用 6767 端口
- OpenCode Server 使用 4096 端口
- WSL 开发环境使用 13001+ 避免冲突

---

## 三、快速启动

### 方式 1: Docker Compose (推荐)

```bash
cd ~/projects/new-api

# 复制环境变量文件
cp .env.example .env

# 编辑 .env，修改端口和数据库配置
nano .env

# 启动服务
docker compose -f docker-compose.dev.yml up -d

# 查看日志
docker compose -f docker-compose.dev.yml logs -f

# 停止服务
docker compose -f docker-compose.dev.yml down
```

### 方式 2: 本地编译运行

```bash
cd ~/projects/new-api

# 编译后端
go build -o new-api

# 启动后端
PORT=13001 ./new-api

# 在另一个终端，启动前端
cd web/default
bun install
bun run dev
```

---

## 四、.env.example 说明

关键环境变量：

| 变量 | 用途 | 示例值 |
|------|------|--------|
| PORT | 服务端口 | 13001 |
| SQL_DSN | 数据库连接 | sqlite://data/new-api.db |
| REDIS_CONN_STRING | Redis 连接 | redis://localhost:16379 |
| LOG_DIR | 日志目录 | ./logs |
| MEMORY_CACHE_ENABLED | 启用内存缓存 | true |
| SYNC_FREQUENCY | 同步频率（秒） | 120 |

**敏感配置**:
- SQL_DSN: 数据库连接字符串
- REDIS_CONN_STRING: Redis 连接字符串
- SESSION_SECRET: 会话密钥（多节点部署必须设置）

**不要提交 .env 到 git！**

---

## 五、本地测试命令

### 健康检查

```bash
curl http://localhost:13001/api/status
```

### 查看日志

```bash
# Docker 方式
docker compose logs -f new-api

# 本地运行方式
tail -f logs/new-api.log
```

### 数据库操作

```bash
# SQLite
sqlite3 data/new-api.db

# PostgreSQL
docker compose exec postgres psql -U root -d new-api

# MySQL
docker compose exec mysql mysql -u root -p new-api
```

---

## 六、常见问题

### 问题 1: 端口被占用

**解决**: 修改 .env 中的 PORT 为其他端口（如 13002）

### 问题 2: 数据库连接失败

**解决**: 
- 检查 SQL_DSN 配置
- 确认数据库服务已启动
- 检查网络连接

### 问题 3: 前端无法访问

**解决**:
- 确认后端已启动
- 检查 FRONTEND_BASE_URL 配置
- 清除浏览器缓存

---

## 七、开发约定

1. **不提交 .env 文件**
2. **不提交真实密钥**
3. **使用测试数据，不导入生产数据**
4. **先本地测试，再容器测试**
5. **提交前运行 lint 和测试**

---

**文档生成完成**
