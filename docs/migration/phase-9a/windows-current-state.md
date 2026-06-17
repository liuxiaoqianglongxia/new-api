# Windows 当前 New API 运行态画像

**生成时间**: 2026-06-15 16:30  
**状态**: 只读摸底完成

---

## 一、项目基本信息

| 项目 | 值 |
|------|-----|
| **项目路径** | `C:\new-api` |
| **WSL 路径** | `/mnt/c/new-api` |
| **项目类型** | Go 编译的预编译二进制（非源码仓库） |
| **Git 仓库** | ❌ 不是 git 仓库 |
| **版本** | v0.0.0 (从 new-api.exe.bak.v0.0.0 推断) |

---

## 二、运行方式

### 启动方式

**方式 1: 直接运行 exe**
```batch
cd C:\new-api
new-api.exe
```

**方式 2: 使用 start.bat 脚本**
```batch
start.bat
```

**方式 3: 注册为 Windows 服务（NSSM）**
```batch
install-service.bat
```

### 启动脚本分析

**start.bat 关键配置**:
```batch
set PORT=3001
set HTTPS_PROXY=http://127.0.0.1:7897
set HTTP_PROXY=http://127.0.0.1:7897
set LOG_DIR=C:\new-api\logs

:: 性能优化
set GIN_MODE=release
set MEMORY_CACHE_ENABLED=true
set SQL_MAX_IDLE_CONNS=20
set SQL_MAX_OPEN_CONNS=100
set SQL_CONN_MAX_LIFETIME=60
set BatchUpdateInterval=300
set GlobalApiRateLimit=0
```

---

## 三、端口监听

| 端口 | 用途 | 状态 |
|------|------|------|
| 3001 | New API Web/API | ✅ 监听中 |

**注意**: 当前使用 3001 端口，不是默认的 3000

---

## 四、数据库

### 数据库类型

**SQLite** (嵌入式数据库)

### 数据库文件

| 文件 | 大小 | 路径 |
|------|------|------|
| one-api.db | 5.0 MB | C:\new-api\one-api.db |
| one-api.db.backup-* | 2.5 MB | C:\new-api\one-api.db.backup-* |

### 数据库配置

```bash
SQL_DSN=  # 空值，表示使用 SQLite
```

**说明**: SQL_DSN 为空时，New API 自动使用 SQLite，数据库文件存放在当前目录

---

## 五、依赖服务

### MySQL

❌ **不依赖**

使用 SQLite，无需 MySQL

### Redis

❌ **不依赖**

未配置 Redis

### 代理服务

✅ **依赖本地代理**

```
HTTPS_PROXY=http://127.0.0.1:7897
HTTP_PROXY=http://127.0.0.1:7897
```

**说明**: 使用本地代理访问外部 API（可能是 Clash 或其他代理工具）

---

## 六、数据目录

| 目录 | 路径 | 大小 | 用途 |
|------|------|------|------|
| logs | C:\new-api\logs | 12 MB | 日志文件 |
| backups | C:\new-api\backups | - | 备份文件 |
| nssm | C:\new-api\nssm | - | Windows 服务管理工具 |

---

## 七、日志目录

**路径**: `C:\new-api\logs`

**日志文件示例**:
```
oneapi-20260612101134.log (1.6K)
oneapi-20260612101200.log (4.9K)
oneapi-20260612101731.log (2.3K)
...
```

**总大小**: 12 MB

---

## 八、Compose / Docker 信息

### Docker Compose

❌ **不存在**

- 没有 docker-compose.yml
- 没有 compose.yaml
- 没有 Dockerfile

### Docker 容器

❌ **未使用 Docker**

当前是直接运行 exe，不是容器化部署

---

## 九、环境变量

### .env 文件

**路径**: `C:\new-api\.env`  
**大小**: 245 bytes

### 环境变量列表

| 变量名 | 用途 | 值 |
|--------|------|------|
| PORT | 服务端口 | `<redacted>` (推断为 3001) |
| SQL_DSN | 数据库连接字符串 | `<redacted>` (空值，使用 SQLite) |
| LOG_DIR | 日志目录 | `<redacted>` (推断为 C:\new-api\logs) |
| MEMORY_CACHE_ENABLED | 启用内存缓存 | `<redacted>` (推断为 true) |
| SYNC_FREQUENCY | 同步频率（秒） | `<redacted>` (推断为 120) |

### start.bat 中的环境变量

| 变量名 | 用途 | 值 |
|--------|------|------|
| PORT | 服务端口 | 3001 |
| HTTPS_PROXY | HTTPS 代理 | http://127.0.0.1:7897 |
| HTTP_PROXY | HTTP 代理 | http://127.0.0.1:7897 |
| LOG_DIR | 日志目录 | C:\new-api\logs |
| GIN_MODE | Gin 框架模式 | release |
| MEMORY_CACHE_ENABLED | 内存缓存 | true |
| SQL_MAX_IDLE_CONNS | 最大空闲连接 | 20 |
| SQL_MAX_OPEN_CONNS | 最大打开连接 | 100 |
| SQL_CONN_MAX_LIFETIME | 连接最大生命周期 | 60 |
| BatchUpdateInterval | 批量更新间隔 | 300 |
| GlobalApiRateLimit | 全局 API 速率限制 | 0 (无限制) |

---

## 十、当前 Windows 服务不要动的原因

### 1. 生产环境运行中

当前 New API 是用户的生产环境，正在提供服务

### 2. 包含真实数据

- 数据库包含真实的渠道配置、token、用户数据
- 日志包含真实的 API 调用记录

### 3. 配置了真实代理

使用了本地代理 (127.0.0.1:7897) 访问外部 API

### 4. 注册为 Windows 服务

可能已通过 NSSM 注册为 Windows 服务，停止会影响系统

### 5. 本轮目标是建立开发副本

WSL 开发副本是独立的，不应该影响 Windows 生产环境

---

## 十一、迁移风险点

### 风险 1: 数据库迁移

**风险**: SQLite 数据库包含真实数据，直接复制可能导致数据不一致

**缓解**: 
- WSL 开发副本使用独立的 SQLite 数据库
- 不导入生产数据
- 使用测试数据

### 风险 2: 端口冲突

**风险**: Windows 使用 3001 端口，WSL 开发副本需要使用不同端口

**缓解**:
- WSL 开发副本使用 13001 或其他端口
- 避免与 Windows 服务冲突

### 风险 3: 代理配置

**风险**: Windows 使用本地代理，WSL 可能需要不同的代理配置

**缓解**:
- WSL 开发副本可以配置不同的代理
- 或者不使用代理（如果可以直接访问）

### 风险 4: 文件路径

**风险**: Windows 使用 C:\new-api\logs，WSL 需要使用 Linux 路径

**缓解**:
- WSL 开发副本使用 ./logs 或 /home/opencode/projects/new-api/logs
- 使用相对路径或环境变量

### 风险 5: 二进制文件

**风险**: Windows exe 文件不能在 WSL 中运行

**缓解**:
- WSL 开发副本需要从源码编译
- 或者使用 Docker 容器

---

## 十二、下一步建议

### 1. Clone 官方仓库

从 GitHub 克隆 New API 官方仓库到 WSL

### 2. 建立开发环境

- 编译 WSL 版本
- 配置独立的 SQLite 数据库
- 配置不同的端口
- 配置测试数据

### 3. 生成部署蓝图

为 wsl-server + 1Panel + Docker Compose 生成部署蓝图

### 4. 执行 Smoke Test

在 WSL 中运行开发副本，验证基本功能

---

**画像生成完成**  
**状态**: 只读摸底完成，未修改任何文件
