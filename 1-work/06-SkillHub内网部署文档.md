# SkillHub 内网部署文档

## 概述

SkillHub 是企业级开源智能体技能注册中心，支持在内网环境私有化部署。本文档基于 Docker Compose 编排所有服务组件，适用于无公网访问的内网环境。

## 架构组件

| 服务 | 说明 | 默认端口 |
|------|------|----------|
| **skillhub-web** | React 前端 + Nginx 反向代理 | 80 |
| **skillhub-server** | Spring Boot 后端 API | 8080 |
| **skill-scanner** | 技能安全扫描服务（可选） | 8000（内部） |
| **postgres** | PostgreSQL 16 数据库 | 5432 |
| **redis** | Redis 7 缓存 | 6379 |

## 前置要求

- Docker 20.10+ 及 Docker Compose V2
- 最低配置：2 核 CPU / 4GB 内存 / 20GB 磁盘
- 推荐配置：4 核 CPU / 8GB 内存 / 50GB+ 磁盘
- 内网镜像仓库（如 Harbor、阿里云 ACR 等）已配置好 SkillHub 镜像

## 第一步：准备镜像

### 方案 A：从公网导出镜像，导入内网

在有公网访问的机器上：

```bash
# 拉取所有需要的镜像
docker pull ghcr.io/iflytek/skillhub-server:latest
docker pull ghcr.io/iflytek/skillhub-web:latest
docker pull ghcr.io/iflytek/skillhub-scanner:latest
docker pull postgres:16-alpine
docker pull redis:7-alpine

# 导出为 tar 文件
docker save -o skillhub-images.tar \
  ghcr.io/iflytek/skillhub-server:latest \
  ghcr.io/iflytek/skillhub-web:latest \
  ghcr.io/iflytek/skillhub-scanner:latest \
  postgres:16-alpine \
  redis:7-alpine
```

将 `skillhub-images.tar` 传输到内网服务器，然后：

```bash
# 在内网服务器上导入
docker load -i skillhub-images.tar

# （可选）推送到内网镜像仓库
docker tag ghcr.io/iflytek/skillhub-server:latest your-registry.example.com/skillhub-server:latest
docker tag ghcr.io/iflytek/skillhub-web:latest your-registry.example.com/skillhub-web:latest
docker tag ghcr.io/iflytek/skillhub-scanner:latest your-registry.example.com/skillhub-scanner:latest
docker push your-registry.example.com/skillhub-server:latest
docker push your-registry.example.com/skillhub-web:latest
docker push your-registry.example.com/skillhub-scanner:latest
```

### 方案 B：使用阿里云镜像（国内环境）

```bash
# runtime.sh 已内置阿里云镜像支持
# 镜像地址：crpi-ptu2rqimrigtq0qx.cn-hangzhou.personal.cr.aliyuncs.com/skill_hub/
```

## 第二步：创建部署目录

```bash
mkdir -p /opt/skillhub
cd /opt/skillhub
```

## 第三步：创建环境配置文件

创建 `.env` 文件：

```bash
cat > .env << 'ENVEOF'
# ============================================================
# SkillHub 内网部署配置
# ============================================================

# ---------- 镜像版本 ----------
# 使用 latest 跟踪稳定版，或指定具体版本如 v0.2.0
SKILLHUB_VERSION=latest

# ---------- 镜像地址 ----------
# 方案1：使用 ghcr.io 官方镜像（需能访问公网或已有缓存）
SKILLHUB_SERVER_IMAGE=ghcr.io/iflytek/skillhub-server
SKILLHUB_WEB_IMAGE=ghcr.io/iflytek/skillhub-web
SKILLHUB_SCANNER_IMAGE=ghcr.io/iflytek/skillhub-scanner
POSTGRES_IMAGE=postgres:16-alpine
REDIS_IMAGE=redis:7-alpine

# 方案2：使用内网镜像仓库（推荐内网部署使用）
# SKILLHUB_SERVER_IMAGE=your-registry.example.com/skillhub-server
# SKILLHUB_WEB_IMAGE=your-registry.example.com/skillhub-web
# SKILLHUB_SCANNER_IMAGE=your-registry.example.com/skillhub-scanner
# POSTGRES_IMAGE=your-registry.example.com/postgres:16-alpine
# REDIS_IMAGE=your-registry.example.com/redis:7-alpine

# ---------- 公网/内网访问地址 ----------
# 必填！改为你的实际访问地址，不带尾部斜杠
SKILLHUB_PUBLIC_BASE_URL=http://skillhub.your-company.internal

# ---------- 端口绑定 ----------
POSTGRES_BIND_ADDRESS=127.0.0.1
POSTGRES_PORT=5432
REDIS_BIND_ADDRESS=127.0.0.1
REDIS_PORT=6379
API_PORT=8080
WEB_PORT=80

# ---------- 数据库 ----------
POSTGRES_DB=skillhub
POSTGRES_USER=skillhub
POSTGRES_PASSWORD=YourStrongPostgresPassword123!

# ---------- 存储配置 ----------
# 内网部署推荐使用 local（本地文件存储），无需额外 S3/MinIO
# 如需 S3/MinIO，改为 s3 并填写下方 S3 配置
SKILLHUB_STORAGE_PROVIDER=local

# --- S3/MinIO 配置（仅当 STORAGE_PROVIDER=s3 时需要） ---
# SKILLHUB_STORAGE_S3_ENDPOINT=http://minio.your-company.internal:9000
# SKILLHUB_STORAGE_S3_PUBLIC_ENDPOINT=
# SKILLHUB_STORAGE_S3_BUCKET=skillhub
# SKILLHUB_STORAGE_S3_ACCESS_KEY=your-access-key
# SKILLHUB_STORAGE_S3_SECRET_KEY=your-secret-key
# SKILLHUB_STORAGE_S3_REGION=cn-default
# SKILLHUB_STORAGE_S3_FORCE_PATH_STYLE=true
# SKILLHUB_STORAGE_S3_AUTO_CREATE_BUCKET=true
# SKILLHUB_STORAGE_S3_PRESIGN_EXPIRY=PT10M

# ---------- 会话安全 ----------
# 内网 HTTP 部署设为 false；HTTPS 部署设为 true
SESSION_COOKIE_SECURE=false

# ---------- 初始管理员账户 ----------
BOOTSTRAP_ADMIN_ENABLED=true
BOOTSTRAP_ADMIN_USER_ID=docker-admin
BOOTSTRAP_ADMIN_USERNAME=admin
BOOTSTRAP_ADMIN_PASSWORD=ChangeMe!2026
BOOTSTRAP_ADMIN_DISPLAY_NAME=Platform Admin
BOOTSTRAP_ADMIN_EMAIL=admin@your-company.com

# ---------- 安全扫描（可选） ----------
# 设为 false 可禁用扫描服务，减少资源占用
SKILLHUB_SECURITY_SCANNER_ENABLED=true

# 扫描服务 LLM 配置（可选，用于 AI 增强扫描）
SKILL_SCANNER_LLM_API_KEY=
SKILL_SCANNER_LLM_BASE_URL=
SKILL_SCANNER_LLM_MODEL=

# ---------- OAuth（可选） ----------
# 内网通常不使用 GitHub OAuth，留空即可
# 如需配置，填写 OAuth 应用的 Client ID 和 Secret
OAUTH2_GITHUB_CLIENT_ID=
OAUTH2_GITHUB_CLIENT_SECRET=

# ---------- 前端 API 代理 ----------
# 通常留空，前端通过 Nginx 反向代理到后端
SKILLHUB_WEB_API_BASE_URL=
SKILLHUB_API_UPSTREAM=http://server:8080

# ---------- 设备认证（可选） ----------
# 默认使用 ${SKILLHUB_PUBLIC_BASE_URL}/device
DEVICE_AUTH_VERIFICATION_URI=
ENVEOF
```

**重要**：请务必修改以下配置项：

| 配置项 | 必须修改 | 说明 |
|--------|----------|------|
| `SKILLHUB_PUBLIC_BASE_URL` | **是** | 改为实际的内网访问地址 |
| `POSTGRES_PASSWORD` | **是** | 数据库密码，使用强密码 |
| `BOOTSTRAP_ADMIN_PASSWORD` | **是** | 管理员初始密码，首次登录后请修改 |
| `SKILLHUB_SERVER_IMAGE` 等 | 看情况 | 如使用内网镜像仓库则需修改 |

## 第四步：创建 docker-compose.yml

```yaml
cat > docker-compose.yml << 'COMPOSEEOF'
services:

  # ==================== 基础设施 ====================

  postgres:
    image: ${POSTGRES_IMAGE:-postgres:16-alpine}
    restart: unless-stopped
    ports:
      - "${POSTGRES_BIND_ADDRESS:-127.0.0.1}:${POSTGRES_PORT:-5432}:5432"
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-skillhub}
      POSTGRES_USER: ${POSTGRES_USER:-skillhub}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-skillhub_demo}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-skillhub} -d ${POSTGRES_DB:-skillhub}"]
      interval: 5s
      timeout: 5s
      retries: 10

  redis:
    image: ${REDIS_IMAGE:-redis:7-alpine}
    restart: unless-stopped
    ports:
      - "${REDIS_BIND_ADDRESS:-127.0.0.1}:${REDIS_PORT:-6379}:6379"
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 10

  # ==================== SkillHub 后端 ====================

  server:
    image: ${SKILLHUB_SERVER_IMAGE:-ghcr.io/iflytek/skillhub-server}:${SKILLHUB_VERSION:-latest}
    restart: unless-stopped
    ports:
      - "${API_PORT:-8080}:8080"
    environment:
      SPRING_PROFILES_ACTIVE: docker
      # 数据库
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/${POSTGRES_DB:-skillhub}
      SPRING_DATASOURCE_USERNAME: ${POSTGRES_USER:-skillhub}
      SPRING_DATASOURCE_PASSWORD: ${POSTGRES_PASSWORD:-skillhub_demo}
      # Redis
      REDIS_HOST: redis
      REDIS_PORT: 6379
      # 会话
      SESSION_COOKIE_SECURE: ${SESSION_COOKIE_SECURE:-false}
      # 公网地址
      SKILLHUB_PUBLIC_BASE_URL: ${SKILLHUB_PUBLIC_BASE_URL:-}
      DEVICE_AUTH_VERIFICATION_URI: ${DEVICE_AUTH_VERIFICATION_URI:-}
      # 存储
      SKILLHUB_STORAGE_PROVIDER: ${SKILLHUB_STORAGE_PROVIDER:-local}
      STORAGE_BASE_PATH: /var/lib/skillhub/storage
      SKILLHUB_STORAGE_S3_ENDPOINT: ${SKILLHUB_STORAGE_S3_ENDPOINT:-}
      SKILLHUB_STORAGE_S3_PUBLIC_ENDPOINT: ${SKILLHUB_STORAGE_S3_PUBLIC_ENDPOINT:-}
      SKILLHUB_STORAGE_S3_BUCKET: ${SKILLHUB_STORAGE_S3_BUCKET:-skillhub}
      SKILLHUB_STORAGE_S3_ACCESS_KEY: ${SKILLHUB_STORAGE_S3_ACCESS_KEY:-}
      SKILLHUB_STORAGE_S3_SECRET_KEY: ${SKILLHUB_STORAGE_S3_SECRET_KEY:-}
      SKILLHUB_STORAGE_S3_REGION: ${SKILLHUB_STORAGE_S3_REGION:-us-east-1}
      SKILLHUB_STORAGE_S3_FORCE_PATH_STYLE: ${SKILLHUB_STORAGE_S3_FORCE_PATH_STYLE:-false}
      SKILLHUB_STORAGE_S3_AUTO_CREATE_BUCKET: ${SKILLHUB_STORAGE_S3_AUTO_CREATE_BUCKET:-false}
      SKILLHUB_STORAGE_S3_PRESIGN_EXPIRY: ${SKILLHUB_STORAGE_S3_PRESIGN_EXPIRY:-PT10M}
      # 安全扫描
      SKILLHUB_SECURITY_SCANNER_ENABLED: ${SKILLHUB_SECURITY_SCANNER_ENABLED:-true}
      SKILLHUB_SECURITY_SCANNER_URL: http://skill-scanner:8000
      SKILLHUB_SECURITY_SCANNER_MODE: upload
      # 管理员
      BOOTSTRAP_ADMIN_ENABLED: ${BOOTSTRAP_ADMIN_ENABLED:-false}
      BOOTSTRAP_ADMIN_USER_ID: ${BOOTSTRAP_ADMIN_USER_ID:-docker-admin}
      BOOTSTRAP_ADMIN_USERNAME: ${BOOTSTRAP_ADMIN_USERNAME:-admin}
      BOOTSTRAP_ADMIN_PASSWORD: ${BOOTSTRAP_ADMIN_PASSWORD:-ChangeMe!2026}
      BOOTSTRAP_ADMIN_DISPLAY_NAME: ${BOOTSTRAP_ADMIN_DISPLAY_NAME:-Admin}
      BOOTSTRAP_ADMIN_EMAIL: ${BOOTSTRAP_ADMIN_EMAIL:-admin@skillhub.local}
      # OAuth
      OAUTH2_GITHUB_CLIENT_ID: ${OAUTH2_GITHUB_CLIENT_ID:-local-placeholder}
      OAUTH2_GITHUB_CLIENT_SECRET: ${OAUTH2_GITHUB_CLIENT_SECRET:-local-placeholder}
    volumes:
      - skillhub_storage:/var/lib/skillhub/storage
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8080/actuator/health"]
      interval: 10s
      timeout: 5s
      retries: 12
      start_period: 60s

  # ==================== SkillHub 前端 ====================

  web:
    image: ${SKILLHUB_WEB_IMAGE:-ghcr.io/iflytek/skillhub-web}:${SKILLHUB_VERSION:-latest}
    restart: unless-stopped
    ports:
      - "${WEB_PORT:-80}:80"
    environment:
      SKILLHUB_API_UPSTREAM: ${SKILLHUB_API_UPSTREAM:-http://server:8080}
      SKILLHUB_WEB_API_BASE_URL: ${SKILLHUB_WEB_API_BASE_URL:-}
      SKILLHUB_PUBLIC_BASE_URL: ${SKILLHUB_PUBLIC_BASE_URL:-}
    depends_on:
      server:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://127.0.0.1/nginx-health"]
      interval: 10s
      timeout: 5s
      retries: 12
      start_period: 10s

  # ==================== 安全扫描（可选） ====================

  skill-scanner:
    image: ${SKILLHUB_SCANNER_IMAGE:-ghcr.io/iflytek/skillhub-scanner}:${SKILLHUB_VERSION:-latest}
    restart: unless-stopped
    environment:
      SKILL_SCANNER_LLM_API_KEY: ${SKILL_SCANNER_LLM_API_KEY:-}
      SKILL_SCANNER_LLM_BASE_URL: ${SKILL_SCANNER_LLM_BASE_URL:-}
      SKILL_SCANNER_LLM_MODEL: ${SKILL_SCANNER_LLM_MODEL:-}
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://127.0.0.1:8000/health"]
      interval: 10s
      timeout: 5s
      retries: 10
    profiles:
      - scanner

volumes:
  postgres_data:
  redis_data:
  skillhub_storage:
COMPOSEEOF
```

> **说明**：安全扫描服务已放入 `profiles: [scanner]`，不会随 `docker compose up` 自动启动。如需启用，参见下方操作说明。

## 第五步：启动服务

### 基础启动（不含安全扫描）

```bash
cd /opt/skillhub
docker compose --env-file .env -f docker-compose.yml up -d
```

### 包含安全扫描的启动

```bash
cd /opt/skillhub
docker compose --env-file .env -f docker-compose.yml --profile scanner up -d
```

### 禁用安全扫描的启动

在 `.env` 中设置：
```
SKILLHUB_SECURITY_SCANNER_ENABLED=false
```

然后正常启动（不加 `--profile scanner`）。

## 第六步：验证部署

### 1. 检查服务状态

```bash
docker compose ps
```

所有服务应为 `healthy` 状态。后端服务 `start_period` 为 60 秒，请耐心等待。

### 2. 访问 Web UI

浏览器打开：`http://<服务器IP>:80`（或 `.env` 中 `WEB_PORT` 指定的端口）

### 3. 验证后端 API

```bash
curl http://localhost:8080/actuator/health
```

应返回 `{"status":"UP"}`。

### 4. 登录管理后台

使用 `.env` 中配置的管理员账户登录：
- 用户名：`admin`
- 密码：`.env` 中 `BOOTSTRAP_ADMIN_PASSWORD` 的值

## 常用运维操作

### 查看日志

```bash
# 查看所有服务日志
docker compose logs -f

# 查看特定服务日志
docker compose logs -f server
docker compose logs -f web
```

### 停止服务

```bash
docker compose down
# 数据保留在 Docker volumes 中，不会丢失
```

### 更新版本

```bash
# 1. 修改 .env 中的 SKILLHUB_VERSION（如改为 v0.3.0）
# 2. 拉取新镜像并重启
docker compose --env-file .env -f docker-compose.yml pull
docker compose --env-file .env -f docker-compose.yml up -d
```

### 完全清理（包括数据）

```bash
# ⚠️ 警告：此操作会删除所有数据！
docker compose down -v
```

### 数据备份

```bash
# 备份 PostgreSQL
docker compose exec postgres pg_dump -U skillhub skillhub > skillhub_backup_$(date +%Y%m%d).sql

# 备份文件存储
docker run --rm -v skillhub_skillhub_storage:/data -v $(pwd):/backup alpine \
  tar czf /backup/skillhub_storage_backup_$(date +%Y%m%d).tar.gz -C /data .
```

### 数据恢复

```bash
# 恢复 PostgreSQL
cat skillhub_backup_YYYYMMDD.sql | docker compose exec -T postgres psql -U skillhub skillhub

# 恢复文件存储
docker run --rm -v skillhub_skillhub_storage:/data -v $(pwd):/backup alpine \
  tar xzf /backup/skillhub_storage_backup_YYYYMMDD.tar.gz -C /data
```

## 内网 Nginx 反向代理（可选）

如需通过域名或 HTTPS 访问，可在内网部署 Nginx 反向代理：

```nginx
server {
    listen 443 ssl;
    server_name skillhub.your-company.internal;

    ssl_certificate     /etc/nginx/ssl/skillhub.crt;
    ssl_certificate_key /etc/nginx/ssl/skillhub.key;

    client_max_body_size 200M;

    location / {
        proxy_pass http://127.0.0.1:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

使用 HTTPS 时，需同步修改 `.env`：
```
SKILLHUB_PUBLIC_BASE_URL=https://skillhub.your-company.internal
SESSION_COOKIE_SECURE=true
```

## 技术栈参考

| 层级 | 技术 |
|------|------|
| 后端 | Java 21 + Spring Boot 3.2.3 |
| 前端 | React 19 + TypeScript + Vite + Tailwind CSS |
| 数据库 | PostgreSQL 16 + Flyway 迁移 |
| 缓存 | Redis 7 |
| 存储 | 本地文件系统 / S3 / MinIO |
| 容器 | Docker + Docker Compose |

## 故障排查

| 问题 | 排查方式 |
|------|----------|
| 后端无法启动 | `docker compose logs server` 检查数据库连接、Redis 连接 |
| 前端无法访问 | 检查 `SKILLHUB_PUBLIC_BASE_URL` 是否正确配置 |
| 上传技能包失败 | 检查存储配置（local 模式检查 volume 权限） |
| 数据库连接失败 | 确认 postgres 服务已 healthy：`docker compose ps postgres` |
| 登录失败 | 确认 `BOOTSTRAP_ADMIN_ENABLED=true`，检查日志中的错误信息 |
