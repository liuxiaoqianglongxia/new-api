# WSL 内部 Docker Engine 安装计划

**生成时间**: 2026-06-15
**Phase**: 9B-preflight-fix-2
**项目**: New API
**状态**: 预检完成，待用户确认安装

---

## 一、为什么选择 WSL 内部 Docker Engine

### 与 Docker Desktop WSL 集成的取舍

| 维度 | Docker Desktop WSL 集成 | WSL 内部 Docker Engine |
|------|------------------------|----------------------|
| 安装复杂度 | 低（Windows 安装即可） | 中（WSL 内 apt 安装） |
| 与 Windows 共享 | ✅ 共享 daemon | ❌ 独立 daemon |
| 贴近生产环境 | ❌ Windows 后台 | ✅ 纯 Linux |
| 与 wsl-server 一致性 | ❌ 不同 | ✅ 相同 |
| 资源占用 | 中（Windows 进程） | 低（仅 WSL 内） |
| systemd 依赖 | 不需要 | 需要启用或手动启动 |
| 许可证 | 商业需付费 | 免费 |

**选择理由**:
1. 未来部署目标是 wsl-server + 1Panel（纯 Linux）
2. WSL 内部 Docker Engine 与生产环境一致
3. 避免 Docker Desktop 商业许可证问题
4. 更贴近 1Panel 的 Docker 管理方式

---

## 二、前置条件

### 2.1 预检结果

| 检查项 | 结果 | 状态 |
|--------|------|------|
| WSL 发行版 | Ubuntu 24.04.4 LTS | ✅ |
| WSL 版本 | WSL2 (kernel 6.6.87.2) | ✅ |
| systemd | offline（未启用） | ⚠️ 需启用或手动启动 |
| cgroup | cgroup2 可用 | ✅ |
| 用户组 | opencode, sudo | ✅ |
| sudo 权限 | 需要密码 | ⚠️ 安装时需输入密码 |
| 磁盘空间 | 949G 可用 | ✅ |
| 内存 | 11Gi total, 2.4Gi available | ⚠️ 偏低但可用 |
| curl | /usr/bin/curl | ✅ |
| gpg | /usr/bin/gpg | ✅ |
| iptables | 未显示 | ⚠️ 可能需要安装 |
| nft | 未安装 | ⚠️ Docker 默认用 iptables |
| Docker 残留 | 无 | ✅ |

### 2.2 需要解决的问题

1. **systemd 未启用**
   - 方案 A: 启用 WSL systemd（推荐）
   - 方案 B: 手动启动 dockerd（备选）

2. **sudo 需要密码**
   - 安装时需要输入用户密码
   - 或配置 sudo nopasswd（不推荐）

3. **iptables 可能缺失**
   - Docker 需要 iptables
   - 安装时会自动处理依赖

---

## 三、安装步骤草案

### 3.1 启用 WSL systemd（推荐）

```bash
# 在 WSL 内执行
sudo nano /etc/wsl.conf
```

添加：
```ini
[boot]
systemd=true
```

然后在 Windows PowerShell 中重启 WSL：
```powershell
wsl --shutdown
```

重新进入 WSL 后验证：
```bash
systemctl is-system-running
# 预期：running
```

### 3.2 安装 Docker Engine

```bash
# 移除旧版本（如有）
sudo apt-get remove docker docker-engine docker.io containerd runc 2>/dev/null || true

# 更新包索引
sudo apt-get update

# 安装依赖
sudo apt-get install -y ca-certificates curl gnupg

# 添加 Docker 官方 GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 添加 Docker 仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 更新包索引
sudo apt-get update

# 安装 Docker Engine
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 验证安装
sudo docker --version
sudo docker compose version
```

### 3.3 配置用户组

```bash
# 将当前用户加入 docker 组
sudo usermod -aG docker $USER

# 刷新组（或重新登录 WSL）
newgrp docker

# 验证（不需要 sudo）
docker --version
docker ps
```

### 3.4 启动 Docker 服务

```bash
# 如果 systemd 已启用
sudo systemctl start docker
sudo systemctl enable docker

# 验证
sudo systemctl status docker
```

---

## 四、安装后验证

### 4.1 版本检查

```bash
docker --version
# 预期：Docker version 2x.x.x, build xxxxxxx

docker compose version
# 预期：Docker Compose version v2.x.x
```

### 4.2 运行测试容器

```bash
docker run hello-world
# 预期：Hello from Docker! 输出
```

### 4.3 检查服务状态

```bash
docker ps
# 预期：空列表（无容器运行）

sudo systemctl status docker
# 预期：active (running)
```

### 4.4 测试 New API dev compose

```bash
cd ~/projects/new-api
docker compose -f docker-compose.dev.yml config
# 预期：输出有效 YAML 配置（不启动）
```

---

## 五、风险与注意事项

### 5.1 systemd/cgroup

- WSL2 默认不启用 systemd
- 需要在 `/etc/wsl.conf` 中启用
- 启用后需要重启 WSL（`wsl --shutdown`）
- cgroup2 已可用，无需额外配置

### 5.2 iptables/nftables

- Docker 默认使用 iptables
- Ubuntu 24.04 可能默认用 nftables
- 安装 Docker 时会自动处理依赖
- 如有冲突，可切换回 iptables-legacy

### 5.3 daemon 自启动

- 启用 systemd 后，Docker 会随系统启动
- 如不需要自启动：`sudo systemctl disable docker`
- 手动启动：`sudo systemctl start docker`

### 5.4 与 Docker Desktop 冲突

- 如果 Windows 已安装 Docker Desktop 并启用 WSL 集成
- 两个 Docker daemon 会冲突
- 解决方案：
  - 禁用 Docker Desktop 的 WSL 集成
  - 或卸载 Docker Desktop
  - 或只使用其中一个

### 5.5 磁盘空间

- Docker 镜像和容器会占用磁盘
- 默认存储在 `/var/lib/docker`
- 当前可用 949G，足够使用
- 定期清理：`docker system prune`

### 5.6 内存

- 当前可用 2.4Gi（总共 11Gi）
- Docker daemon 本身占用约 200-500MB
- New API dev 环境（PostgreSQL + Redis + New API）约需 1-2Gi
- 建议：关闭不必要的应用，或增加 WSL 内存限制

---

## 六、回滚方案

### 6.1 停止 Docker

```bash
sudo systemctl stop docker
sudo systemctl disable docker
```

### 6.2 卸载 Docker

```bash
sudo apt-get purge -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo apt-get autoremove -y
```

### 6.3 清理数据

```bash
# 删除 Docker 数据（镜像、容器、卷）
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd

# 删除配置
sudo rm -rf /etc/docker
sudo rm -f /etc/apt/keyrings/docker.gpg
sudo rm -f /etc/apt/sources.list.d/docker.list
```

### 6.4 恢复用户组

```bash
sudo gpasswd -d $USER docker
newgrp docker
```

---

## 七、安装前必须用户确认

### 确认清单

- [ ] 理解为什么选择 WSL 内部 Docker Engine
- [ ] 理解需要启用 systemd
- [ ] 理解需要输入 sudo 密码
- [ ] 理解如果 Windows 有 Docker Desktop 需要处理冲突
- [ ] 理解安装后会占用约 1-2Gi 磁盘空间
- [ ] 理解安装后 Docker 会随系统启动（除非禁用）
- [ ] 确认当前 WSL 内存足够（建议至少 4Gi available）

### 确认方式

用户回复以下任一即可：
- "确认安装 Docker Engine"
- "可以安装"
- "开始安装"

---

## 八、安装后下一步

1. 验证 Docker 安装成功
2. 测试 `docker compose -f docker-compose.dev.yml config`
3. 启动 New API dev 环境：`docker compose -f docker-compose.dev.yml up -d`
4. 验证健康检查：`curl http://localhost:13001/api/status`
5. 生成 Phase 9B 完成报告

---

## 九、参考文档

- Docker 官方 Ubuntu 安装指南：https://docs.docker.com/engine/install/ubuntu/
- WSL systemd 支持：https://devblogs.microsoft.com/wsl/systemd-support-is-now-available-in-wsl/
- New API Phase 9B run plan：`~/projects/new-api/docs/ops/phase-9b-wsl-dev-run-plan.md`

---

**文档版本**: v1.0
**最后更新**: 2026-06-15
**状态**: 待用户确认安装
