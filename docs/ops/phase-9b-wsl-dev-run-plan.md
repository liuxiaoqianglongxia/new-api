# Phase 9B: WSL Dev Environment Run Plan

**生成时间**: 2026-06-15
**Phase**: 9B
**项目**: New API
**环境**: WSL opencode
**Docker 方案**: WSL 内部 Docker Engine（方案 B）

---

## 一、前置条件

### 1.1 Docker 可用性

**当前状态**: ❌ Docker 未安装在 WSL opencode

**已选择方案**: WSL 内部 Docker Engine（方案 B）

**选择理由**:
- 贴近 wsl-server + 1Panel 生产环境
- 纯 Linux，与部署目标一致
- 避免 Docker Desktop 商业许可证问题

**安装计划**: `~/projects/new-api/docs/ops/wsl-docker-engine-install-plan.md`

**安装前预检结果**:
- ✅ Ubuntu 24.04.4 LTS, WSL2
- ✅ cgroup2 可用
- ✅ 用户在 sudo 组
- ⚠️ systemd 未启用（需启用）
- ⚠️ sudo 需要密码
- ✅ 磁盘 949G 可用
- ⚠️ 内存 2.4Gi available（偏低但可用）

**安装前必须用户确认**: 详见安装计划文档第七节

### 1.2 wsl-server Discovery

**当前状态**: ⏳ 待用户在 wsl-server 上执行 discovery 脚本

**执行包路径**: `~/.config/opencode/references/wsl-server-discovery-copy-paste.md`

**下一步**: 用户执行后更新 deployment-topology.md

### 1.3 New API 项目文件

**当前状态**: ✅ 就绪

| 文件 | 状态 | 说明 |
|------|------|------|
| docker-compose.dev.yml | ✅ 存在 | 端口已修改为 13001:3000 |
| docker-compose.yml | ✅ 存在 | 生产用 |
| .env.example | ✅ 存在 | PORT=13001（正确） |
| .env | ✅ 存在 | 不读取内容 |
| AGENTS.md | ✅ 存在 | 188 行增强版 |
| docs/ | ✅ 已恢复 | 14 个上游文件已恢复 |

---

## 二、端口规划

### 2.1 当前冲突

| 端口 | 服务 | 状态 |
|------|------|------|
| 3000 | Appsmith | Windows 已占用 |
| 3001 | Windows New API | Windows 已占用 |
| 3002 | OpenChamber | 全局已占用 |
| 4096 | OpenCode Server | 全局已占用 |
| 6767 | Paseo | 全局已占用 |
| **13001** | **WSL New API dev** | **目标端口** |

### 2.2 docker-compose.dev.yml 端口配置

**当前配置**（已修改）：
```yaml
ports:
  - "13001:3000"  # 宿主 13001 → 容器 3000（避免与 Appsmith 3000 冲突）
```

**修改状态**: ✅ 已完成

### 2.3 内部服务端口

| 服务 | 容器内端口 | 宿主端口 | 说明 |
|------|-----------|---------|------|
| New API | 3000 | 13001 | 映射到宿主 |
| PostgreSQL | 5432 | 不暴露 | 仅容器内部 |
| Redis | 6379 | 不暴露 | 仅容器内部 |

---

## 三、数据库选择

### 3.1 Dev 环境推荐

**PostgreSQL**（docker-compose.dev.yml 已配置）

理由：
- 与生产环境一致（wsl-server 将使用 PostgreSQL）
- docker-compose.dev.yml 已包含 PostgreSQL 15
- 支持完整 SQL 特性

### 3.2 备选方案

**SQLite**（最简单）
- 无需额外容器
- 配置：`SQL_DSN=sqlite://data/new-api.db`
- 适合快速测试

**MySQL**（备选）
- 配置：`SQL_DSN=mysql://root:password@mysql:3306/new-api`
- 需要修改 compose 文件

### 3.3 禁止事项

- ❌ 不导入 Windows one-api.db
- ❌ 不复制 Windows .env 真实值
- ❌ 不使用生产数据库

---

## 四、Redis 配置

### 4.1 当前配置（docker-compose.dev.yml）

```yaml
redis:
  image: redis:7-alpine
  container_name: new-api-dev-redis
  networks:
    - dev-network
```

### 4.2 连接字符串

```
REDIS_CONN_STRING=redis://redis
```

### 4.3 端口暴露

**默认不暴露宿主端口**（仅容器内部通信）

如需调试暴露：
```yaml
ports:
  - "16379:6379"  # 宿主 16379 → 容器 6379
```

---

## 五、启动步骤

### 5.0 Docker 安装（前置）

**必须先完成 Docker Engine 安装**，详见：
`~/projects/new-api/docs/ops/wsl-docker-engine-install-plan.md`

安装步骤摘要：
1. 启用 WSL systemd（`/etc/wsl.conf` 添加 `[boot] systemd=true`）
2. 重启 WSL（`wsl --shutdown`）
3. 安装 Docker Engine（`sudo apt install docker-ce docker-ce-cli ...`）
4. 配置用户组（`sudo usermod -aG docker $USER`）
5. 验证安装（`docker --version && docker run hello-world`）

**安装前必须用户确认**。

### 5.1 前置准备

```bash
cd ~/projects/new-api

# 确认 .env 存在（如不存在则复制）
if [ ! -f .env ]; then
  cp .env.example .env
fi

# 编辑 .env，确认端口（如需要）
nano .env
# 确认 PORT=13001
```

**注意**: docker-compose.dev.yml 端口已修改为 13001:3000，无需再改。

### 5.2 启动服务

```bash
# 启动（后台）
docker compose -f docker-compose.dev.yml up -d

# 查看日志
docker compose -f docker-compose.dev.yml logs -f

# 等待健康检查通过（约 10-30 秒）
```

### 5.3 验证启动

```bash
# 检查容器状态
docker compose -f docker-compose.dev.yml ps

# 健康检查
curl http://localhost:13001/api/status

# 预期输出：{"code":0,"data":true,"message":""}
```

### 5.4 前端开发（可选）

```bash
cd web/default
bun install
bun run dev
# 前端 dev server: http://localhost:5173
# API 代理到 http://localhost:13001
```

---

## 六、健康检查

### 6.1 API 健康检查

```bash
curl http://localhost:13001/api/status
```

**预期输出**:
```json
{"code":0,"data":true,"message":""}
```

### 6.2 容器健康检查

```bash
docker compose -f docker-compose.dev.yml ps
```

**预期输出**:
```
NAME                STATUS
new-api-dev         Up (healthy)
new-api-dev-pg      Up (healthy)
new-api-dev-redis   Up
```

---

## 七、停止步骤

### 7.1 正常停止

```bash
docker compose -f docker-compose.dev.yml down
```

### 7.2 停止并删除数据（重置）

```bash
docker compose -f docker-compose.dev.yml down -v
```

⚠️ 这会删除所有数据卷（dev_data, dev_pg_data）

---

## 八、日志查看

### 8.1 实时日志

```bash
docker compose -f docker-compose.dev.yml logs -f
```

### 8.2 指定服务日志

```bash
docker compose -f docker-compose.dev.yml logs -f new-api
docker compose -f docker-compose.dev.yml logs -f postgres
docker compose -f docker-compose.dev.yml logs -f redis
```

### 8.3 最近 100 行

```bash
docker compose -f docker-compose.dev.yml logs --tail=100
```

---

## 九、回滚

### 9.1 代码回滚

```bash
cd ~/projects/new-api
git checkout docker-compose.dev.yml
git checkout .env
```

### 9.2 数据回滚

```bash
# 停止并删除所有数据
docker compose -f docker-compose.dev.yml down -v

# 重新启动（空数据库）
docker compose -f docker-compose.dev.yml up -d
```

### 9.3 完全回滚

```bash
# 停止所有容器
docker compose -f docker-compose.dev.yml down -v

# 删除镜像
docker rmi new-api-dev:local

# 清理未使用的镜像
docker image prune -f
```

---

## 十、禁止事项

### 10.1 绝对禁止

- ❌ 不动 Windows New API（C:\new-api）
- ❌ 不复制 Windows .env 真实值
- ❌ 不导入 Windows one-api.db
- ❌ 不部署到 wsl-server
- ❌ 不修改 wsl-server
- ❌ 不 push / PR / release

### 10.2 需要确认

- ⚠️ 安装 Docker（需要用户确认方案 A 或 B）
- ⚠️ 修改 docker-compose.dev.yml 端口（需要用户确认）
- ⚠️ 暴露 Redis 端口到宿主（需要用户确认）

### 10.3 允许

- ✅ 在 WSL opencode 内启动 dev 环境
- ✅ 使用 .env.example 创建 .env
- ✅ 使用空数据库测试
- ✅ 本地开发调试

---

## 十一、故障排查

### 11.1 端口被占用

**症状**: `Error: bind: address already in use`

**解决**:
```bash
# 检查端口占用
ss -tlnp | grep 13001

# 修改 .env 中的 PORT
nano .env
# 改为 PORT=13002
```

### 11.2 Docker daemon 未启动

**症状**: `Cannot connect to the Docker daemon`

**解决**:
```bash
# Docker Desktop WSL 集成：确保 Windows Docker Desktop 运行中
# WSL Docker Engine：
sudo systemctl start docker
```

### 11.3 权限不足

**症状**: `permission denied while trying to connect to the Docker daemon socket`

**解决**:
```bash
sudo usermod -aG docker $USER
newgrp docker
# 或重新登录
```

### 11.4 数据库连接失败

**症状**: `dial tcp: lookup postgres: no such host`

**解决**:
```bash
# 检查 PostgreSQL 容器是否运行
docker compose -f docker-compose.dev.yml ps postgres

# 查看 PostgreSQL 日志
docker compose -f docker-compose.dev.yml logs postgres
```

---

## 十二、下一步

### Phase 9B 正式启动前

1. ✅ 确认 Docker 安装方案（已选择方案 B: WSL 内部 Docker Engine）
2. ✅ docker-compose.dev.yml 端口已修改为 13001:3000
3. ⏳ 用户确认安装 Docker Engine（详见安装计划）
4. ⏳ 用户在 wsl-server 执行 discovery 脚本
5. ⏳ 更新 deployment-topology.md

### Phase 9B 正式执行

1. 安装 Docker Engine（用户确认后执行安装计划）
2. 验证 Docker 安装：`docker --version && docker run hello-world`
3. 启动 dev 环境：`docker compose -f docker-compose.dev.yml up -d`
4. 验证健康检查：`curl http://localhost:13001/api/status`
5. 前端开发测试（可选）
6. 生成 Phase 9B 完成报告

---

**文档版本**: v1.0
**最后更新**: 2026-06-15
