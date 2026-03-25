# DevOps 平台学习计划

> 创建日期: 2026-03-25
> 目标: 快速上手 DevOps 平台设计与实施

---

## 学习路线图

```
阶段一（1-2周）                      阶段二（1-2周）
    ┌──────────────┐                     ┌──────────────┐
    │  Linux 基础   │ ──────▶             │   Docker      │
    │  网络基础     │                     │   Compose     │
    └──────────────┘                     └──────────────┘
                                              │
                                              ▼
                                  ┌──────────────────────┐
                                  │   阶段三（3-4周）      │
                                  │   Kubernetes         │
                                  │   基础 + 进阶         │
                                  └──────────────────────┘
                                              │
                      ┌───────────────────────┼───────────────────────┐
                      ▼                       ▼                       ▼
              ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
              │  阶段四       │      │  阶段五       │      │  阶段六       │
              │  CI/CD       │      │  监控日志     │      │  网络高可用    │
              └──────────────┘      └──────────────┘      └──────────────┘
                      │                       │                       │
                      └───────────────────────┼───────────────────────┘
                                              ▼
                                  ┌──────────────────────┐
                                  │   阶段七（1周）        │
                                  │   安全与认证          │
                                  └──────────────────────┘
                                              │
                                              ▼
                                  ┌──────────────────────┐
                                  │   阶段八（1周）        │
                                  │   Rancher 管理平台    │
                                  └──────────────────────┘
```

---

## 阶段一：基础准备（1-2周）

### 1. Linux 基础

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| 常用命令 | ls, cd, cp, mv, rm, grep, find, awk, sed | 在虚拟机/云服务器上练习 |
| 用户权限 | chmod, chown, sudo, /etc/passwd | 创建用户并设置权限 |
| 服务管理 | systemctl, journalctl | 安装并管理 nginx 服务 |
| 文件系统 | mount, df, du, ln | 挂载磁盘、创建软链接 |
| 网络基础 | ip, netstat, ssh, iptables | 配置静态IP、SSH远程登录 |
| Shell 脚本 | 变量、循环、条件判断、函数 | 编写自动备份脚本 |

**练习任务**：
- [ ] 安装 CentOS/Ubuntu 虚拟机
- [ ] 创建新用户并配置 sudo 权限
- [ ] 安装 nginx 服务并设置为开机自启
- [ ] 编写一个每天自动备份指定目录的脚本

**推荐资源**：
- [鸟哥的Linux私房菜](https://linux.vbird.org/)
- [Linux Journey](https://linuxjourney.com/)

---

### 2. 网络基础

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| TCP/IP 协议 | 三次握手、四次挥手、端口 | 使用 Wireshark 抓包分析 |
| DNS 解析 | A记录、CNAME、域名解析流程 | 配置本地 hosts 文件 |
| HTTP/HTTPS | 状态码、Header、请求方法 | 使用 curl 测试API |
| 负载均衡原理 | L4/L7负载均衡 | 配置 HAProxy |

**练习任务**：
- [ ] 使用 curl 访问网站并分析响应头
- [ ] 配置本地 hosts 解析自定义域名
- [ ] 使用 Wireshark 抓取 HTTP 请求

---

## 阶段二：容器技术（1-2周）

### 3. Docker

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| 基础命令 | run, ps, exec, logs, stop, rm | 运行 nginx、redis 容器 |
| 镜像操作 | pull, push, build, tag | 编写 Dockerfile 构建 Web 应用 |
| 数据卷 | -v, --mount | 持久化数据库数据 |
| 网络 | bridge, host, none, 自定义网络 | 连接多个容器 |
| Docker Compose | docker-compose.yml | 编排 Web+DB+Redis 三层架构 |
| 镜像优化 | 多阶段构建、缓存优化 | 减小镜像体积 |

**练习任务**：
- [ ] 运行一个 nginx 容器并访问
- [ ] 编写 Dockerfile 构建一个 Python/Java Web 应用
- [ ] 使用 Docker Compose 部署 WordPress + MySQL
- [ ] 优化应用镜像大小（目标 < 100MB）

**推荐资源**：
- [Docker 官方文档](https://docs.docker.com/)
- [Docker — 从入门到实践](https://yeasy.gitbook.io/docker_practice/)

---

## 阶段三：Kubernetes 核心（3-4周）

### 4. Kubernetes 基础

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| 集群搭建 | Minikube/Kind | 在本地搭建 K8s 集群 |
| 核心概念 | Pod, Deployment, Service, Namespace | 部署一个简单的 Web 应用 |
| 配置管理 | ConfigMap, Secret | 配置应用的配置文件和密钥 |
| 存储卷 | PV, PVC, StorageClass | 为数据库配置持久化存储 |
| 健康检查 | livenessProbe, readinessProbe | 配置应用健康检查 |
| 资源限制 | requests, limits | 限制 Pod 的 CPU/内存 |

**练习任务**：
- [ ] 使用 Minikube 搭建本地 K8s 集群
- [ ] 部署一个 nginx Deployment 并通过 Service 暴露
- [ ] 使用 ConfigMap 管理 nginx 配置
- [ ] 为应用配置 livenessProbe 和 readinessProbe

---

### 5. Kubernetes 进阶

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| Ingress | 入口控制器、路由规则 | 配置域名访问应用 |
| HPA | 自动扩缩容 | 基于 CPU 指标自动扩容应用 |
| DaemonSet | 守护进程集 | 在每个节点部署日志采集组件 |
| StatefulSet | 有状态应用 | 部署一个 Redis 集群 |
| RBAC | 角色权限控制 | 创建只读用户和开发用户 |
| NetworkPolicy | 网络策略 | 配置命名空间网络隔离 |

**练习任务**：
- [ ] 配置 Ingress 实现域名访问应用
- [ ] 部署 HPA，基于 CPU 使用率自动扩容
- [ ] 使用 RBAC 配置只读用户
- [ ] 配置 NetworkPolicy 实现命名空间隔离

**推荐资源**：
- [Kubernetes 官方文档](https://kubernetes.io/zh-cn/docs/)
- [Kubernetes Handbook](https://kubernetes.feisky.xyz/)

---

## 阶段四：CI/CD（1-2周）

### 6. Jenkins

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| Pipeline | Declarative Pipeline | 编写 Jenkinsfile |
| 凭证管理 | Credentials | 存储git仓库和docker仓库凭证 |
| 插件系统 | Git, Docker, Kubernetes | 安装并配置常用插件 |
| 触发器 | Webhook, 定时触发 | 配置 Git Push 自动触发构建 |
| 多环境部署 | dev/test/prod | 分环境部署流程 |

**练习任务**：
- [ ] 使用 Docker 部署 Jenkins
- [ ] 编写 Jenkinsfile 实现 Git → Build → Test 流程
- [ ] 配置 Gitee Webhook 自动触发构建
- [ ] 实现多环境部署（dev/test/prod）

---

### 7. 镜像构建

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| Dockerfile 最佳实践 | 多阶段构建、缓存优化 | 优化应用镜像大小 |
| Harbor | 镜像仓库、项目权限 | 搭建私有镜像仓库 |
| Kaniko | 容器内构建 | 使用 Kaniko 替代 Docker-in-Docker |

**练习任务**：
- [ ] 使用 Docker Compose 部署 Harbor
- [ ] 配置 Harbor 项目权限
- [ ] 使用 Kaniko 在 K8s 中构建镜像

---

## 阶段五：监控与日志（1-2周）

### 8. Prometheus

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| 指标采集 | Exporter、PromQL | 监控服务器和应用指标 |
| 告警规则 | AlertManager | 配置 CPU 高负载告警 |
| 服务发现 | K8s SD | 自动发现 K8s 中的服务 |

**练习任务**：
- [ ] 部署 Prometheus 并监控 K8s 集群
- [ ] 配置应用使用 Exporter 暴露指标
- [ ] 编写 PromQL 查询语句
- [ ] 配置告警规则

---

### 9. Grafana

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| 数据源配置 | Prometheus、Loki | 连接数据源 |
| 仪表盘 | Panel、Query | 创建集群监控大盘 |
| 组织与权限 | Team、Folder | 配置团队访问权限 |

**练习任务**：
- [ ] 部署 Grafana 并连接 Prometheus
- [ ] 创建 K8s 集群监控仪表盘
- [ ] 配置组织和用户权限

---

### 10. 日志系统

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| Promtail | 日志采集、标签添加 | 采集应用日志 |
| Loki | 日志存储、查询 | 使用 LogQL 查询日志 |
| 日志权限 | 命名空间隔离 | 配置团队只能看自己日志 |

**练习任务**：
- [ ] 部署 Loki + Promtail
- [ ] 采集应用日志到 Loki
- [ ] 使用 LogQL 查询和分析日志
- [ ] 配置日志保留策略

---

## 阶段六：网络与高可用（1周）

### 11. 网络与负载均衡

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| Calico CNI | Pod网络、NetworkPolicy | 配置命名空间网络隔离 |
| HAProxy | 四层/七层负载均衡 | 为 K8s API Server 配置 LB |
| Keepalived | VIP 高可用 | 实现 Master 节点 VIP 漂移 |
| DNS | 内网域名解析 | 配置 .local 域名解析 |

**练习任务**：
- [ ] 搭建 HAProxy + Keepalived 实现高可用
- [ ] 配置内网 DNS 服务器
- [ ] 部署 Calico 并配置 NetworkPolicy

---

## 阶段七：安全与认证（1周）

### 12. 安全与认证

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| HTTPS | 自签名证书、CA | 创建内部CA并签发证书 |
| CAS 协议 | 单点登录原理 | 理解 CAS 认证流程 |
| RBAC | K8s 角色权限 | 配置团队开发权限 |
| Secret 管理 | 敏感数据存储 | 加密存储数据库密码 |

**练习任务**：
- [ ] 创建内部 CA 并签发服务证书
- [ ] 配置 K8s Secret 加密
- [ ] 理解 CAS 单点登录流程
- [ ] 配置 RBAC 实现多租户权限隔离

---

## 阶段八：管理平台（1周）

### 13. Rancher

| 知识点 | 学习重点 | 练手场景 |
|--------|----------|----------|
| 集群管理 | 多集群导入 | 使用 Rancher 管理 K8s |
| 应用商店 | Helm Chart | 一键部署常用应用 |
| 用户管理 | 全局/集群/项目角色 | 配置团队成员权限 |

**练习任务**：
- [ ] 部署 Rancher
- [ ] 导入现有 K8s 集群到 Rancher
- [ ] 使用 Rancher 部署应用
- [ ] 配置用户和团队权限

---

## 学习资源推荐

| 资源 | 说明 | 链接 |
|------|------|------|
| Katacoda | 在线实验环境，免费练习 | https://www.katacoda.com/ |
| Play with K8s | 免费 K8s 练习环境 | https://labs.play-with-k8s.com/ |
| Play with Docker | 免费 Docker 练习环境 | https://labs.play-with-docker.com/ |
| Minikube | 本地单节点 K8s 集群 | https://minikube.sigs.k8s.io/ |
| Kind | 本地多节点 K8s 集群 | https://kind.sigs.k8s.io/ |
| K8s 官方文档 | 最权威的 K8s 文档 | https://kubernetes.io/zh-cn/docs/ |
| CNCF 云原生路线图 | 了解完整的云原生生态 | https://github.com/cncf/landscape |
| Docker 官方文档 | Docker 学习 | https://docs.docker.com/ |
| Jenkins 官方文档 | Jenkins 学习 | https://www.jenkins.io/doc/ |

---

## 学习进度跟踪

### 阶段一：基础准备
- [ ] Linux 基础
- [ ] 网络基础

### 阶段二：容器技术
- [ ] Docker 基础
- [ ] Docker Compose

### 阶段三：Kubernetes 核心
- [ ] K8s 基础
- [ ] K8s 进阶

### 阶段四：CI/CD
- [ ] Jenkins
- [ ] 镜像构建

### 阶段五：监控与日志
- [ ] Prometheus
- [ ] Grafana
- [ ] 日志系统

### 阶段六：网络与高可用
- [ ] 网络与负载均衡

### 阶段七：安全与认证
- [ ] 安全与认证

### 阶段八：管理平台
- [ ] Rancher

---

## 学习建议

1. **边学边练**：每学一个概念都要动手实践，不要只看不练
2. **记录笔记**：把踩过的坑、重要的命令记录下来
3. **参与社区**：遇到问题多查 GitHub Issues 和 Stack Overflow
4. **从小到大**：先在本地用 Minikube/Kind 练习，再尝试多节点集群
5. **循序渐进**：不要试图一次性掌握所有内容，按阶段稳步前进

---

## 学习周期总览

| 阶段 | 内容 | 周期 |
|------|------|------|
| 一 | 基础准备 | 1-2周 |
| 二 | 容器技术 | 1-2周 |
| 三 | Kubernetes 核心 | 3-4周 |
| 四 | CI/CD | 1-2周 |
| 五 | 监控与日志 | 1-2周 |
| 六 | 网络与高可用 | 1周 |
| 七 | 安全与认证 | 1周 |
| 八 | 管理平台 | 1周 |
| **总计** | | **10-14周** |

---

*最后更新: 2026-03-25*
