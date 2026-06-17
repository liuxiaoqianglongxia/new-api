# 运维手册 (Runbook)

**生成时间**: 2026-06-15  
**项目**: New API

---

## 一、启动

### Docker Compose 启动

```bash
cd ~/projects/new-api

# 启动所有服务
docker compose up -d

# 查看状态
docker compose ps

# 查看日志
docker compose logs -f
```

### 本地启动

```bash
cd ~/projects/new-api

# 编译
go build -o new-api

# 启动（实际端口 13002，原规划 13001 被 WSL 系统 socket 占用）
PORT=13002 ./new-api
```

---

## 二、停止

### Docker Compose 停止

```bash
# 停止所有服务
docker compose down

# 停止并删除数据卷（谨慎！）
docker compose down -v
```

### 本地停止

```bash
# 查找进程
ps aux | grep new-api

# 停止进程
kill <pid>
```

---

## 三、查看日志

### Docker 日志

```bash
# 查看所有服务日志
docker compose logs

# 查看特定服务日志
docker compose logs new-api
docker compose logs postgres
docker compose logs redis

# 实时跟踪日志
docker compose logs -f new-api

# 查看最近 100 行
docker compose logs --tail=100 new-api
```

### 本地日志

```bash
# 查看日志文件
tail -f logs/new-api.log

# 搜索错误
grep "ERROR" logs/new-api.log
```

---

## 四、备份

### 完整备份

```bash
#!/bin/bash
BACKUP_DIR="/backup/new-api/$(date +%Y%m%d-%H%M%S)"
mkdir -p $BACKUP_DIR

# 备份配置文件
cp docker-compose.yml $BACKUP_DIR/
cp .env $BACKUP_DIR/

# 备份数据库（PostgreSQL）
docker exec new-api-postgres pg_dump -U postgres newapi > $BACKUP_DIR/postgres-dump.sql

# 备份应用数据
tar -czf $BACKUP_DIR/new-api-data.tar.gz -C data/new-api .

echo "Backup completed: $BACKUP_DIR"
```

### 数据库备份

```bash
# PostgreSQL
docker exec new-api-postgres pg_dump -U postgres newapi > backup-$(date +%Y%m%d).sql

# MySQL
docker exec new-api-mysql mysqldump -u root -p newapi > backup-$(date +%Y%m%d).sql

# SQLite
cp data/new-api.db backup-$(date +%Y%m%d).db
```

---

## 五、恢复

### 从备份恢复

```bash
# 停止服务
docker compose down

# 恢复数据库
docker compose up -d postgres
sleep 5
docker exec -i new-api-postgres psql -U postgres newapi < backup/postgres-dump.sql

# 恢复应用数据
tar -xzf backup/new-api-data.tar.gz -C data/new-api/

# 恢复配置文件
cp backup/docker-compose.yml .
cp backup/.env .

# 重启服务
docker compose up -d
```

---

## 六、常见故障

### 故障 1: 服务无法启动

**症状**: docker compose up 失败

**排查**:
```bash
# 查看详细日志
docker compose logs

# 检查端口占用
netstat -tlnp | grep 3000

# 检查磁盘空间
df -h
```

**解决**:
- 修改 PORT 为其他端口
- 清理磁盘空间
- 检查配置文件语法

---

### 故障 2: 数据库连接失败

**症状**: 日志显示 "database connection failed"

**排查**:
```bash
# 检查数据库服务
docker compose ps postgres

# 测试连接
docker exec -it new-api-postgres psql -U postgres -c "SELECT 1"
```

**解决**:
- 检查 SQL_DSN 配置
- 确认数据库服务已启动
- 检查密码是否正确

---

### 故障 3: Redis 连接失败

**症状**: 日志显示 "redis connection failed"

**排查**:
```bash
# 检查 Redis 服务
docker compose ps redis

# 测试连接
docker exec -it new-api-redis redis-cli -a your_password ping
```

**解决**:
- 检查 REDIS_CONN_STRING 配置
- 确认 Redis 服务已启动
- 检查密码是否正确

---

### 故障 4: API 返回 500 错误

**症状**: API 请求返回 500

**排查**:
```bash
# 查看错误日志
docker compose logs new-api | grep "ERROR"

# 检查数据库状态
docker exec -it new-api-postgres psql -U postgres newapi -c "SELECT COUNT(*) FROM channels"
```

**解决**:
- 检查错误日志
- 确认数据库正常
- 检查渠道配置

---

### 故障 5: 内存占用过高

**症状**: 内存使用持续增长

**排查**:
```bash
# 查看资源使用
docker stats new-api

# 检查缓存配置
docker exec -it new-api env | grep CACHE
```

**解决**:
- 调整 MEMORY_CACHE_ENABLED
- 减少 SQL_MAX_OPEN_CONNS
- 重启服务释放内存

---

## 七、性能优化

### 数据库优化

```bash
# PostgreSQL
docker exec -it new-api-postgres psql -U postgres newapi -c "VACUUM ANALYZE"

# 调整连接池
SQL_MAX_IDLE_CONNS=20
SQL_MAX_OPEN_CONNS=100
```

### 缓存优化

```bash
# 启用内存缓存
MEMORY_CACHE_ENABLED=true

# 调整同步频率
SYNC_FREQUENCY=120
```

### 日志优化

```bash
# 禁用错误日志（减少 IO）
ERROR_LOG_ENABLED=false

# 使用日志轮转
docker compose logs --follow | rotatelogs /var/log/new-api/%Y%m%d.log 86400
```

---

## 八、监控

### 健康检查

```bash
# API 健康检查
curl http://localhost:3000/api/status

# 数据库检查
docker exec -it new-api-postgres psql -U postgres newapi -c "SELECT 1"
```

### 指标监控

```bash
# 启用 pprof
ENABLE_PPROF=true

# 访问 pprof
curl http://localhost:3000/debug/pprof/
```

---

## 九、升级

### Docker 镜像升级

```bash
# 拉取新镜像
docker compose pull

# 重启服务
docker compose up -d

# 查看日志确认
docker compose logs -f new-api
```

### 回滚

```bash
# 指定旧版本镜像
# 修改 docker-compose.yml 中的 image 标签
image: calciumion/new-api:v1.0.0

# 重启
docker compose up -d
```

---

## 十、安全建议

1. **定期更新镜像**
2. **修改默认密码**
3. **使用 HTTPS**
4. **限制访问 IP**
5. **定期备份**
6. **监控异常日志**

---

**运维手册生成完成**
