# wsl-server + 1Panel 部署蓝图

**生成时间**: 2026-06-15  
**项目**: New API  
**目标环境**: wsl-server (本地服务器)

---

## 一、部署架构

```
wsl-server (本地服务器)
├── 1Panel (容器编排管理面板)
│   ├── new-api (API 网关)
│   ├── postgres / mysql (数据库)
│   └── redis (缓存)
└── 反向代理 (Nginx/Caddy)
    └── https://api.example.com → new-api:3000
```

---

## 二、推荐目录结构

### 1Panel 标准应用目录

```
/opt/1panel/apps/custom/new-api/
├── docker-compose.yml
├── .env
├── data/
│   ├── new-api/        # New API 数据
│   ├── postgres/       # PostgreSQL 数据
│   └── redis/          # Redis 数据
└── logs/
    └── new-api/        # 应用日志
```

### 备选：用户自定义目录

```
/home/user/projects/new-api-deploy/
├── docker-compose.yml
├── .env
└── ...
```

---

## 三、Docker Compose 方案

### 生产环境 Compose 文件

```yaml
version: '3.8'

services:
  new-api:
    image: calciumion/new-api:latest
    container_name: new-api
    restart: always
    command: --log-dir /app/logs
    ports:
      - "3000:3000"  # 只暴露 Web/API 端口
    volumes:
      - ./data/new-api:/data
      - ./logs/new-api:/app/logs
    environment:
      - SQL_DSN=postgresql://postgres:${POSTGRES_PASSWORD}@postgres:5432/newapi
      - REDIS_CONN_STRING=redis://:${REDIS_PASSWORD}@redis:6379
      - TZ=Asia/Shanghai
      - ERROR_LOG_ENABLED=true
      - BATCH_UPDATE_ENABLED=true
      - NODE_NAME=new-api-prod
      - SESSION_SECRET=${SESSION_SECRET}  # 必须设置随机字符串
    depends_on:
      - postgres
      - redis
    networks:
      - new-api-network
    healthcheck:
      test: ["CMD-SHELL", "wget -q -O - http://localhost:3000/api/status | grep -o '\"success\":\\s*true' || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3

  redis:
    image: redis:7-alpine
    container_name: new-api-redis
    restart: always
    command: ["redis-server", "--requirepass", "${REDIS_PASSWORD}"]
    volumes:
      - ./data/redis:/data
    networks:
      - new-api-network

  postgres:
    image: postgres:15-alpine
    container_name: new-api-postgres
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: newapi
    volumes:
      - ./data/postgres:/var/lib/postgresql/data
    networks:
      - new-api-network

networks:
  new-api-network:
    driver: bridge
```

---

## 四、数据库选择建议

### 推荐：PostgreSQL

**优点**:
- 性能优秀
- 功能丰富
- 社区活跃
- 适合生产环境

**配置**:
```bash
SQL_DSN=postgresql://postgres:your_password@postgres:5432/newapi
```

### 备选：MySQL

**优点**:
- 广泛使用
- 工具丰富

**配置**:
```bash
SQL_DSN=mysql://root:your_password@mysql:3306/newapi
```

### 开发环境：SQLite

**优点**:
- 无需额外服务
- 简单易用

**缺点**:
- 不适合高并发
- 不适合多节点部署

**配置**:
```bash
SQL_DSN=
SQLITE_PATH=/data/new-api.db
```

---

## 五、端口规划

| 服务 | 容器端口 | 宿主端口 | 说明 |
|------|---------|---------|------|
| new-api | 3000 | 3000 | Web/API 端口 |
| postgres | 5432 | 不暴露 | 内部网络 |
| redis | 6379 | 不暴露 | 内部网络 |

**注意**: 
- 只暴露 new-api 的 3000 端口
- 数据库和 Redis 只在内部网络
- 通过反向代理访问 API

---

## 六、数据卷规划

| 服务 | 数据卷 | 宿主路径 | 用途 |
|------|--------|---------|------|
| new-api | /data | ./data/new-api | 应用数据 |
| new-api | /app/logs | ./logs/new-api | 应用日志 |
| postgres | /var/lib/postgresql/data | ./data/postgres | 数据库数据 |
| redis | /data | ./data/redis | Redis 数据 |

---

## 七、备份规划

### 备份内容

1. **Compose 文件**: docker-compose.yml
2. **环境变量**: .env
3. **数据库**: PostgreSQL dump
4. **应用数据**: data/new-api/
5. **日志**: logs/new-api/ (可选)

### 备份脚本

```bash
#!/bin/bash
# backup-new-api.sh

BACKUP_DIR="/backup/new-api/$(date +%Y%m%d-%H%M%S)"
mkdir -p $BACKUP_DIR

# 备份 Compose 和 .env
cp /opt/1panel/apps/custom/new-api/docker-compose.yml $BACKUP_DIR/
cp /opt/1panel/apps/custom/new-api/.env $BACKUP_DIR/

# 备份数据库
docker exec new-api-postgres pg_dump -U postgres newapi > $BACKUP_DIR/postgres-dump.sql

# 备份应用数据
tar -czf $BACKUP_DIR/new-api-data.tar.gz -C /opt/1panel/apps/custom/new-api/data/new-api .

echo "Backup completed: $BACKUP_DIR"
```

### 备份频率

- **数据库**: 每天一次
- **应用数据**: 每周一次
- **配置文件**: 每次修改后

---

## 八、回滚规划

### 回滚步骤

1. **停止当前服务**:
   ```bash
   docker compose down
   ```

2. **恢复数据库**:
   ```bash
   docker compose up -d postgres
   docker exec -i new-api-postgres psql -U postgres newapi < backup/postgres-dump.sql
   ```

3. **恢复应用数据**:
   ```bash
   tar -xzf backup/new-api-data.tar.gz -C /opt/1panel/apps/custom/new-api/data/new-api/
   ```

4. **恢复配置文件**:
   ```bash
   cp backup/docker-compose.yml /opt/1panel/apps/custom/new-api/
   cp backup/.env /opt/1panel/apps/custom/new-api/
   ```

5. **重启服务**:
   ```bash
   docker compose up -d
   ```

### 镜像回滚

如果需要回滚镜像版本：

```bash
# 修改 docker-compose.yml 中的镜像标签
image: calciumion/new-api:v1.0.0  # 指定版本

# 重启服务
docker compose pull
docker compose up -d
```

---

## 九、1Panel 操作步骤

### 步骤 1: 安装 1Panel

```bash
# 参考官方文档
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh
sudo bash quick_start.sh
```

### 步骤 2: 创建编排

1. 登录 1Panel Web UI
2. 进入"容器" → "编排"
3. 点击"创建编排"
4. 填写名称：new-api
5. 粘贴 docker-compose.yml 内容
6. 点击"提交"

### 步骤 3: 配置环境变量

1. 进入编排详情
2. 编辑 .env 文件
3. 设置：
   - POSTGRES_PASSWORD
   - REDIS_PASSWORD
   - SESSION_SECRET
4. 保存

### 步骤 4: 启动服务

```bash
# 在 1Panel 中点击"启动"
# 或使用命令行
cd /opt/1panel/apps/custom/new-api
docker compose up -d
```

### 步骤 5: 查看日志

```bash
# 在 1Panel 中查看
# 或使用命令行
docker compose logs -f new-api
```

### 步骤 6: 停止/回滚

```bash
# 停止
docker compose down

# 回滚（见第八节）
```

---

## 十、安全建议

1. **修改默认密码**:
   - POSTGRES_PASSWORD
   - REDIS_PASSWORD
   - SESSION_SECRET

2. **使用 HTTPS**:
   - 配置反向代理（Nginx/Caddy）
   - 申请 SSL 证书

3. **限制访问**:
   - 配置防火墙
   - 使用 IP 白名单

4. **定期备份**:
   - 自动化备份脚本
   - 异地备份

5. **监控告警**:
   - 配置健康检查
   - 设置告警通知

---

## 十一、监控建议

### 健康检查

```bash
curl http://localhost:3000/api/status
```

### 日志监控

```bash
docker compose logs -f new-api | grep "ERROR"
```

### 资源监控

```bash
docker stats new-api
```

---

**部署蓝图生成完成**  
**状态**: 待执行（需要用户提供 wsl-server 访问方式）
