# DevOps 平台设计方案

> 创建日期: 2026-03-18
> 状态: 待审批
> 版本: v1.2

## 1. 项目背景

在内网环境下的Linux服务器集群（10-50台）上部署一套完整的DevOps平台，服务于50-200人的开发团队。

## 2. 需求概述

| 需求项 | 描述 |
|--------|------|
| Gitee集成 | push → 代码审查 → 合并主分支 → 自动部署 |
| 自动扩缩容 | 基于CPU/内存使用率自动扩缩容 |
| 容器日志 | 提供容器日志查看功能，按命名空间权限隔离 |
| CAS认证 | 集成现有Apereo CAS单点登录系统 |
| 离线部署 | 完全内网环境，需支持离线安装 |

### 2.1 环境约束

- **网络环境**: 完全内网，无外网访问
- **服务器规模**: 10-50台Linux服务器
- **应用类型**: Web应用、微服务、定时任务、数据处理
- **用户规模**: 50-200人
- **高可用级别**: 开发环境级别（允许偶尔停机）

## 3. 技术选型

### 3.1 核心组件

| 组件 | 选型 | 版本建议 | 说明 |
|------|------|----------|------|
| 容器运行时 | Docker | 24.x | 容器运行环境 |
| 容器编排 | Kubernetes | 1.28+ | kubeadm部署，3 Master高可用 |
| 负载均衡 | HAProxy + Keepalived | - | Master VIP高可用 |
| CI/CD | Jenkins | 2.440+ | Kubernetes Plugin动态Pod构建 |
| 镜像构建 | Kaniko | v1.19+ | 无特权容器内构建镜像 |
| 镜像仓库 | Harbor | 2.10+ | 私有镜像仓库 |
| 日志收集 | Loki | 2.9+ | 轻量级日志系统 |
| 日志采集 | Promtail | 2.9+ | DaemonSet部署 |
| 监控 | Prometheus | 2.48+ | 指标采集 |
| 指标服务 | Metrics Server | 0.7+ | HPA依赖的指标服务 |
| 可视化 | Grafana | 10.x | 统一可视化界面 |
| 网络插件 | Calico | 3.26+ | 支持NetworkPolicy |
| Ingress | Nginx Ingress | 1.9+ | 入口控制器 |
| DNS | CoreDNS | 1.11+ | 集群内DNS（K8s自带） |
| 内网DNS | Bind9/CoreDNS | - | 内部域名解析 |
| 证书管理 | 自签名CA | - | 内网HTTPS证书 |
| CAS代理 | cas-auth-proxy | - | 自建CAS认证代理 |
| K8s管理平台 | Rancher | 2.8+ | 多集群管理、权限控制、应用商店 |

### 3.2 选型理由

**选择Kubernetes而非Docker Swarm/K3s的原因**:
- 社区支持完善，问题容易找到解决方案
- HPA自动扩缩容成熟稳定
- 生态丰富，组件集成度高

**选择Loki而非ELK的原因**:
- 资源占用低（约2GB内存 vs ELK的8GB+）
- 与K8s集成简单（Promtail原生支持）
- 与Prometheus/Grafana生态统一
- 运维复杂度低

**选择Kaniko而非Docker-in-Docker的原因**:
- 无需特权模式，安全性更高
- 专为容器内构建镜像设计
- 避免Docker socket挂载的安全风险

**CAS认证集成方案**:
OAuth2 Proxy原生不支持CAS协议，因此采用自建cas-auth-proxy方案：
- 基于nginx + lua-cas 或 cas-overlay 构建认证代理
- 或使用 Keycloak 作为中间层（支持CAS协议）
- 本方案采用 **nginx + lua-resty-cas** 自建代理

## 4. 架构设计

### 4.1 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        内网环境                                  │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  离线资源服务器                           │    │
│  │  • RPM/DEB 软件包                                        │    │
│  │  • Docker 镜像包 (tar)                                   │    │
│  │  • Helm Charts (ChartMuseum)                             │    │
│  │  • Jenkins 插件离线包                                    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                   │
│  ┌───────────────────────────┼───────────────────────────────┐  │
│  │                    访问入口层                               │  │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │  │
│  │  │ HAProxy     │    │ Keepalived  │    │ 内网DNS     │   │  │
│  │  │ (负载均衡)   │    │ (VIP)       │    │ (域名解析)   │   │  │
│  │  └─────────────┘    └─────────────┘    └─────────────┘   │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Kubernetes 集群                        │    │
│  │                                                          │    │
│  │   ┌─────────────────────────────────────────────────┐    │    │
│  │   │              Nginx Ingress Controller            │    │    │
│  │   │              + cas-auth-proxy (认证)             │    │    │
│  │   └─────────────────────────────────────────────────┘    │    │
│  │                              │                            │    │
│  │   ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    │    │
│  │   │ Rancher │  │ Harbor  │  │ Jenkins │  │ 业务应用 │    │    │
│  │   │ (管理)  │  │ (镜像)  │  │ (CI/CD) │  │         │    │    │
│  │   └─────────┘  └─────────┘  └─────────┘  └─────────┘    │    │
│  │                                                          │    │
│  │   ┌─────────┐  ┌─────────┐                               │    │
│  │   │ Loki    │  │ Prometheus│                              │    │
│  │   │ (日志)  │  │ + Grafana│                              │    │
│  │   └─────────┘  └─────────┘                               │    │
│  │                                                          │    │
│  │   ┌─────────────────────────────────────────────────┐    │    │
│  │   │ Prometheus + Grafana + Metrics Server + HPA     │    │    │
│  │   └─────────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌───────────────┐                    ┌───────────────┐         │
│  │  Gitee        │                    │   CAS系统     │         │
│  │  (内网私有化)  │                    │   (现有)      │         │
│  └───────────────┘                    └───────────────┘         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 集群节点规划

以20台服务器为例：

| 节点角色 | 数量 | 配置建议 | 运行组件 |
|----------|------|----------|----------|
| Master | 3 | 4C8G, SSD 100GB | kube-apiserver, etcd, controller-manager, scheduler |
| 负载均衡 | 2 | 2C4G | HAProxy + Keepalived (可复用Master或独立部署) |
| 基础设施 | 3 | 8C16G, SSD 500GB | Jenkins, Harbor, Prometheus, Grafana, Loki |
| Worker | 12+ | 8C32G, SSD 200GB | 业务应用Pod |

**节点标签设计**:
- Master: `node-role.kubernetes.io/control-plane=`
- 基础设施: `node-role.kubernetes.io/infra=`
- Worker: `node-role.kubernetes.io/worker=`

### 4.3 命名空间规划

| 命名空间 | 用途 | 说明 |
|----------|------|------|
| kube-system | 系统组件 | K8s核心组件 |
| infra | 基础设施 | Jenkins, Harbor等 |
| monitoring | 监控日志 | Prometheus, Grafana, Loki |
| auth | 认证服务 | cas-auth-proxy |
| dev | 开发环境 | 开发环境应用 |
| test | 测试环境 | 测试环境应用 |
| prod | 生产环境 | 生产环境应用 |
| team-* | 团队命名空间 | 按团队划分（可选） |

### 4.4 网络规划

```
内网网段: 10.0.0.0/8 (根据实际情况调整)

├── 节点网络:     10.10.0.0/16     (服务器物理IP)
├── VIP:          10.10.0.100      (Master高可用VIP)
├── Pod网络:      10.244.0.0/16    (Calico分配)
├── Service网络:  10.96.0.0/16     (ClusterIP)
└── Ingress:      使用节点网络IP或VIP
```

### 4.5 内网DNS配置

需要在内网DNS服务器上配置以下解析：

```
# 内网域名解析示例
10.10.0.100    k8s-api.local        # K8s API Server VIP
10.10.0.101    harbor.local         # Harbor服务
10.10.0.102    jenkins.local        # Jenkins服务
10.10.0.103    grafana.local        # Grafana服务
10.10.0.104    gitee.local          # Gitee服务
10.10.0.105    cas.local            # CAS服务
*.app.local    10.10.0.100          # 业务应用泛域名
```

### 4.6 存储方案

| 存储类 | 类型 | 用途 | 说明 |
|--------|------|------|------|
| local-path | 本地SSD | 高性能需求 | 数据库、etcd等，Pod无法迁移 |
| nfs-client | NFS网络存储 | 共享访问 | 配置文件、日志归档 |
| ceph-rbd | Ceph块存储 | 高可用存储 | 可选，生产环境推荐 |

**组件存储需求**:

| 组件 | 存储类 | 大小 | 说明 |
|------|--------|------|------|
| etcd | local-path | 10GB × 3 | Master本地SSD |
| Jenkins | local-path/nfs | 200GB | 构建缓存、工作空间 |
| Rancher | local-path | 20GB | 管理平台数据 |
| Harbor | local-path | 500GB | 镜像存储（建议大容量） |
| Loki | local-path | 200GB | 日志存储（配保留策略） |
| Prometheus | local-path | 100GB | 监控数据（配保留策略） |
| 备份存储 | NFS | 1TB+ | etcd备份、数据备份 |

## 5. 功能模块设计

### 5.1 CI/CD 流水线

**流程图**:

```
开发者 Push → Gitee 代码审查 → 合并主分支 → Webhook触发
                                              ↓
                                        Jenkins Pipeline
                                              ↓
    Checkout → Build → Test → Kaniko Build → Push to Harbor → Deploy to K8s
```

**Gitee Webhook 配置**:

在Gitee项目设置中配置Webhook：
- URL: `http://jenkins.local/gitee-webhook/`
- 密码: 设置webhook密码
- 触发事件: Push、Pull Request合并

**Jenkins Gitee 插件配置**:

1. 安装 Gitee Plugin（离线安装 .hpi 文件）
2. 系统配置中添加 Gitee 服务器信息
3. 配置 Webhook URL 和密码

**Jenkinsfile 模板（使用Kaniko）**:

```groovy
pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          serviceAccountName: jenkins-deployer
          containers:
          - name: builder
            image: harbor.local/devops/maven:3.9-jdk17
            command: ['sleep', '99d']
          - name: kaniko
            image: harbor.local/devops/executor:v1.19.0
            command: ['sleep', '99d']
            volumeMounts:
            - name: docker-config
              mountPath: /kaniko/.docker
          volumes:
          - name: docker-config
            configMap:
              name: docker-config
          '''
    }
  }

  environment {
    HARBOR_URL = 'harbor.local'
    APP_NAME = "${env.JOB_NAME}"
    IMAGE_TAG = "${env.BUILD_NUMBER}"
    NAMESPACE = "${env.ENV_NAMESPACE ?: 'dev'}"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: env.BRANCH_NAME ?: 'main',
            url: 'http://gitee.local/group/repo.git',
            credentialsId: 'gitee-credentials'
      }
    }

    stage('Build') {
      steps {
        container('builder') {
          sh 'mvn clean package -DskipTests'
        }
      }
    }

    stage('Test') {
      steps {
        container('builder') {
          sh 'mvn test'
        }
      }
    }

    stage('Kaniko Build & Push') {
      steps {
        container('kaniko') {
          sh """
            /kaniko/executor \
              --dockerfile=Dockerfile \
              --context=. \
              --destination=${HARBOR_URL}/apps/${APP_NAME}:${IMAGE_TAG} \
              --destination=${HARBOR_URL}/apps/${APP_NAME}:latest
          """
        }
      }
    }

    stage('Deploy to K8s') {
      steps {
        container('builder') {
          sh """
            kubectl set image deployment/${APP_NAME} \\
              ${APP_NAME}=${HARBOR_URL}/apps/${APP_NAME}:${IMAGE_TAG} \\
              -n ${NAMESPACE}
          """
        }
      }
    }
  }
}
```

**Jenkins RBAC 配置**:

```yaml
# jenkins-rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-deployer
  namespace: infra
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-deployer
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-deployer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins-deployer
subjects:
- kind: ServiceAccount
  name: jenkins-deployer
  namespace: infra
```

**多环境部署策略**:

| 分支 | 部署环境 | 触发方式 | ENV_NAMESPACE |
|------|----------|----------|---------------|
| feature/* | 不部署 | - | - |
| develop | dev | 自动 | dev |
| release/* | test | 自动 | test |
| main | prod | 手动审批 | prod |

### 5.2 自动扩缩容 (HPA)

**Metrics Server 部署**:

```yaml
# metrics-server.yaml (离线环境需修改镜像地址)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - name: metrics-server
        image: harbor.local/library/metrics-server:v0.7.0
        args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
```

**扩缩容指标**:
- CPU使用率 > 70% 触发扩容
- 内存使用率 > 80% 触发扩容
- CPU使用率 < 30% 触发缩容

**HPA 配置模板**:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Pods
        value: 1
        periodSeconds: 300
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
```

**扩缩容策略建议**:

| 应用类型 | 最小副本 | 最大副本 | CPU阈值 | 内存阈值 |
|----------|----------|----------|---------|----------|
| 核心Web服务 | 3 | 20 | 60% | 70% |
| 一般Web服务 | 2 | 10 | 70% | 80% |
| 后台任务 | 1 | 5 | 80% | 80% |
| 数据处理 | 2 | 8 | 75% | 85% |

### 5.3 日志系统

**架构**:

```
Pod日志 → Promtail (DaemonSet) → Loki → Grafana
                ↓
         添加K8s元数据标签
         (namespace, app, pod)
```

**Loki 保留策略配置**:

```yaml
# loki-config.yaml
limits_config:
  retention_period: 168h  # 7天
  max_query_length: 721h
  max_query_parallelism: 32

compactor:
  working_directory: /data/loki/compactor
  shared_store: filesystem
  retention_enabled: true
  retention_delete_delay: 2h
  retention_delete_worker_count: 150

schema_config:
  configs:
  - from: 2024-01-01
    store: boltdb-shipper
    object_store: filesystem
    schema: v12
    index:
      prefix: index_
      period: 24h
```

**查询能力**:
- 按应用名称过滤: `{app="my-app"}`
- 按命名空间过滤: `{namespace="dev"}`
- 时间范围查询
- 实时日志追踪: 类似 `tail -f`
- 日志聚合统计

**权限隔离**:
- Grafana 通过 CAS SSO 认证
- 使用 Grafana 的文件夹和团队权限控制
- 配置 Loki 数据源的查询过滤

### 5.4 CAS 认证集成

**方案说明**:

由于 OAuth2 Proxy 原生不支持 CAS 协议，采用以下方案：

1. **Jenkins**: 使用官方 CAS Plugin
2. **Grafana**: 使用 Auth Proxy 模式，通过 cas-auth-proxy 代理
3. **Harbor**: 配置 OIDC（CAS 6.x 支持 OIDC）或使用 Auth Proxy
4. **业务应用**: 统一通过 cas-auth-proxy 代理访问

**cas-auth-proxy 部署（基于 OpenResty + lua-resty-cas）**:

```yaml
# cas-auth-proxy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cas-auth-proxy
  namespace: auth
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cas-auth-proxy
  template:
    spec:
      containers:
      - name: cas-auth-proxy
        image: harbor.local/library/cas-auth-proxy:1.0
        ports:
        - containerPort: 8080
        env:
        - name: CAS_SERVER_URL
          value: "https://cas.local"
        - name: CAS_SERVICE_URL
          value: "https://app.local"
        - name: UPSTREAM_URL
          value: "http://app-service:8080"
---
apiVersion: v1
kind: Service
metadata:
  name: cas-auth-proxy
  namespace: auth
spec:
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: cas-auth-proxy
```

**Ingress 集成 CAS 认证示例**:

```yaml
# 需要CAS认证的应用Ingress配置
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-with-cas
  namespace: dev
  annotations:
    nginx.ingress.kubernetes.io/auth-url: "http://cas-auth-proxy.auth.svc.cluster.local:8080/auth"
    nginx.ingress.kubernetes.io/auth-signin: "https://cas.local/login?service=$scheme://$host$request_uri"
spec:
  ingressClassName: nginx
  rules:
  - host: app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 8080
```

**各组件CAS集成方式**:

| 组件 | 集成方式 | 配置说明 |
|------|----------|----------|
| Jenkins | CAS Plugin | 安装 cas-plugin.hpi，配置CAS服务器地址 |
| Grafana | Auth Proxy | 通过Ingress annotation或配置auth.proxy |
| Harbor | OIDC | CAS 6.x配置OIDC，或使用Auth Proxy |
| 业务应用 | Ingress Auth | 通过Ingress annotation统一认证 |
| K8s Dashboard | Auth Proxy | 通过cas-auth-proxy代理访问 |
| Rancher | 内置认证 | 配置与CAS/OIDC集成 |

### 5.6 Rancher 管理平台

**功能特性**：
- 多集群统一管理
- 可视化工作负载编辑
- 用户权限管理（RBAC）
- 应用商店（Helm Chart）
- 监控告警集成
- 日志管理集成

**部署架构**：

```
┌─────────────────────────────────────────────────────────────────┐
│                      Rancher 部署架构                            │
└─────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────┐
                    │    Rancher Server   │
                    │    (infra命名空间)   │
                    │    2副本高可用       │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
       ┌────────────┐   ┌────────────┐   ┌────────────┐
       │ Cluster A  │   │ Cluster B  │   │ Cluster C  │
       │ (生产)     │   │ (测试)     │   │ (开发)     │
       └────────────┘   └────────────┘   └────────────┘
```

**离线部署步骤**：

```bash
# 1. 准备Rancher离线镜像
# 从外网下载 rancher-images.tar
docker load -i rancher-images.tar

# 2. 推送到内网Harbor
docker tag rancher/rancher:v2.8.0 harbor.local/rancher/rancher:v2.8.0
docker push harbor.local/rancher/rancher:v2.8.0

# 3. 添加Helm仓库（离线环境需搭建ChartMuseum）
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable

# 4. 安装cert-manager（Rancher依赖）
kubectl apply -f cert-manager.yaml

# 5. Helm安装Rancher
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --create-namespace \
  --set hostname=rancher.local \
  --set replicas=2 \
  --set rancherImage=harbor.local/rancher/rancher \
  --set systemDefaultRegistry=harbor.local
```

**Rancher资源配置**：

```yaml
# rancher-values.yaml
hostname: rancher.local
replicas: 2

# 使用内网镜像仓库
rancherImage: harbor.local/rancher/rancher
systemDefaultRegistry: harbor.local

# 资源限制
resources:
  requests:
    cpu: 500m
    memory: 1Gi
  limits:
    cpu: 2
    memory: 4Gi

# TLS配置（使用自签名证书）
tls: external
```

**CAS/OIDC认证集成**：

Rancher支持多种认证方式，可与现有CAS系统集成：

1. **方式一：OIDC集成**（CAS 6.x支持OIDC）
   - Rancher → 配置 → 认证 → OpenLDAP/OIDC
   - 配置CAS的OIDC端点信息

2. **方式二：通过Keycloak中转**
   - Keycloak配置CAS身份提供者
   - Rancher对接Keycloak OIDC

3. **方式三：使用Rancher本地认证**
   - 管理员手动创建用户
   - 适合小规模团队

**权限管理**：

```
Rancher权限模型：
├── Global Roles（全局角色）
│   ├── Administrator（管理员）
│   ├── Standard User（普通用户）
│   └── User-Base（基础用户）
├── Cluster Roles（集群角色）
│   ├── Cluster Owner（集群所有者）
│   ├── Cluster Member（集群成员）
│   └── View Cluster（只读）
└── Project Roles（项目角色）
    ├── Project Owner（项目所有者）
    ├── Project Member（项目成员）
    └── Read Only（只读）
```

**内网DNS配置**：

```
10.10.0.100    rancher.local        # Rancher管理界面
```

### 5.5 证书管理

**自签名CA方案**:

1. 创建内部CA证书
2. 为各服务签发证书
3. 将CA证书分发到所有节点和客户端

```bash
# 创建CA证书
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt \
  -subj "/C=CN/ST=Beijing/L=Beijing/O=Company/CN=Internal CA"

# 签发服务证书示例
openssl genrsa -out harbor.local.key 2048
openssl req -new -key harbor.local.key -out harbor.local.csr \
  -subj "/C=CN/ST=Beijing/L=Beijing/O=Company/CN=harbor.local"
openssl x509 -req -in harbor.local.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out harbor.local.crt -days 365 -sha256
```

**证书分发**:
- 将 `ca.crt` 复制到所有节点的 `/etc/pki/ca-trust/source/anchors/`
- 执行 `update-ca-trust` 更新信任库
- Docker需要配置 `insecure-registries` 或信任CA

## 6. 离线部署方案

### 6.1 需要准备的离线资源

| 资源类型 | 内容 | 大小估算 |
|----------|------|----------|
| 系统包 | Docker、kubelet、kubeadm、kubectl及依赖 | ~1GB |
| 基础镜像 | pause、coredns、etcd、calico、nginx-ingress等 | ~5GB |
| 组件镜像 | Jenkins、Harbor、Prometheus全家桶、Loki、Grafana、Kaniko、Rancher等 | ~35GB |
| Helm Charts | 各组件的Helm包 | ~300MB |
| Jenkins插件 | 常用插件离线包 | ~800MB |
| 系统依赖 | conntrack、socat、ipvsadm等 | ~100MB |
| **总计** | | **~42GB** |

### 6.2 离线安装流程

```
外网环境                              内网环境
┌─────────────┐                  ┌─────────────────┐
│ 1. 下载所有  │ ── U盘/传输 ───▶ │ 3. 部署到离线    │
│    离线资源  │                  │    资源服务器    │
│ 2. 打包      │                  │ 4. 各节点安装    │
└─────────────┘                  └─────────────────┘
```

### 6.3 离线资源清单

```
offline-resources/
├── packages/
│   ├── centos7/
│   │   ├── docker-ce/           # Docker RPM包
│   │   ├── kubernetes/          # K8s RPM包
│   │   └── dependencies/        # 系统依赖包
│   ├── ubuntu2004/
│   │   ├── docker-ce/           # Docker DEB包
│   │   ├── kubernetes/          # K8s DEB包
│   │   └── dependencies/        # 系统依赖包
│   └── ...
├── images/
│   ├── k8s-images.tar           # K8s基础镜像
│   ├── calico-images.tar        # Calico镜像
│   ├── nginx-ingress-images.tar
│   ├── metrics-server.tar
│   ├── jenkins-images.tar       # Jenkins + agent镜像
│   ├── harbor-images.tar        # Harbor全部镜像
│   ├── loki-stack.tar           # Loki + Promtail
│   ├── prometheus-stack.tar     # Prometheus + Grafana
│   ├── kaniko.tar
│   ├── cas-auth-proxy.tar
│   └── rancher-images.tar       # Rancher全套镜像
├── charts/
│   └── (Helm Charts压缩包)
├── plugins/
│   └── jenkins-plugins/
│       ├── cas-plugin.hpi
│       ├── kubernetes.hpi
│       ├── git.hpi
│       ├── gitee.hpi
│       └── ...
├── certs/
│   └── internal-ca/             # 内部CA证书
└── scripts/
    ├── load-images.sh           # 加载镜像脚本
    ├── retag-images.sh          # 重新打Tag脚本
    ├── install-docker.sh        # Docker安装脚本
    ├── install-k8s.sh           # K8s安装脚本
    └── setup-harbor.sh          # Harbor初始化脚本
```

### 6.4 离线镜像加载脚本

```bash
#!/bin/bash
# load-images.sh

IMAGE_DIR="/opt/offline-resources/images"
HARBOR_URL="harbor.local"

# 加载所有镜像
for tar_file in ${IMAGE_DIR}/*.tar; do
  echo "Loading $tar_file..."
  docker load -i $tar_file
done

echo "All images loaded successfully."

# 重新打Tag并推送到Harbor（需要先部署Harbor）
# ./retag-images.sh
```

```bash
#!/bin/bash
# retag-images.sh

HARBOR_URL="harbor.local"

# 镜像重命名映射
declare -A IMAGE_MAP=(
  ["k8s.gcr.io/pause:3.9"]="${HARBOR_URL}/library/pause:3.9"
  ["k8s.gcr.io/coredns/coredns:v1.11.1"]="${HARBOR_URL}/library/coredns:v1.11.1"
  ["docker.io/kindest/kindnetd:v20230511-dc714da8"]="${HARBOR_URL}/library/kindnetd:v20230511"
  # ... 添加其他镜像映射
)

for src in "${!IMAGE_MAP[@]}"; do
  dest="${IMAGE_MAP[$src]}"
  echo "Tagging $src -> $dest"
  docker tag "$src" "$dest"
  docker push "$dest"
done

echo "All images retagged and pushed to Harbor."
```

## 7. 部署顺序

### 阶段一: 基础设施准备 (1-2天)

1. 准备离线资源包
2. 部署离线资源服务器（HTTP/NFS）
3. 配置内网DNS
4. 生成并分发内部CA证书
5. 在所有节点安装 Docker
6. 部署 HAProxy + Keepalived（Master VIP）
7. 使用 kubeadm 初始化集群
8. 安装 Calico 网络插件
9. 安装 Metrics Server
10. 安装 Nginx Ingress

### 阶段二: 存储与镜像仓库 (1天)

1. 部署 NFS 服务器（或使用现有）
2. 部署 local-path-provisioner
3. 部署 Harbor 镜像仓库
4. 推送所有镜像到 Harbor
5. 配置 Docker 信任 Harbor 证书

### 阶段三: CI/CD (1-2天)

1. 部署 Jenkins
2. 安装 Jenkins 插件（离线）
3. 配置 Jenkins CAS 认证
4. 配置 Gitee Webhook
5. 创建 Jenkins ServiceAccount 和 RBAC
6. 测试构建流水线

### 阶段四: 管理平台 (1天)

1. 部署 cert-manager（Rancher依赖）
2. 部署 Rancher
3. 导入K8s集群到Rancher
4. 配置Rancher认证（对接CAS或OIDC）
5. 创建项目和用户权限
6. 配置应用商店

### 阶段五: 监控日志 (1天)

1. 部署 Prometheus + Grafana
2. 部署 Loki + Promtail
3. 配置 Grafana CAS 集成
4. 配置日志权限隔离
5. 配置告警规则

### 阶段六: 认证与业务对接 (1天)

1. 部署 cas-auth-proxy
2. 创建命名空间和 ResourceQuota
3. 创建应用部署模板
4. 测试完整 CI/CD 流程
5. 测试 CAS 认证流程

## 8. 安全配置

### 8.1 网络策略

**命名空间基础隔离**:

```yaml
# 允许同命名空间通信 + DNS查询 + 监控采集
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-allow-internal
  namespace: dev
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector: {}
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: monitoring
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: infra
  egress:
  - to:
    - podSelector: {}
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
```

### 8.2 RBAC 配置

**团队开发者角色示例**:

```yaml
# 团队开发者角色（仅限自己命名空间）
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: team-a
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]
- apiGroups: ["networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["get", "list", "watch", "create", "update", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: team-a-developers
  namespace: team-a
subjects:
- kind: Group
  name: team-a-developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

### 8.3 资源配额

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-a-quota
  namespace: team-a
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    pods: "50"
    services: "20"
    secrets: "50"
    configmaps: "50"
    persistentvolumeclaims: "20"
```

### 8.4 Secret 管理

- 使用 K8s Secret 存储敏感信息
- 配置 Secret 加密（EncryptionConfiguration）
- 定期轮换敏感凭据
- 考虑使用 Sealed Secrets 或 External Secrets（可选）

## 9. 备份与恢复

### 9.1 etcd 备份

```bash
#!/bin/bash
# etcd-backup.sh

ETCD_ENDPOINTS="https://10.10.0.10:2379,https://10.10.0.11:2379,https://10.10.0.12:2379"
ETCD_CACERT="/etc/kubernetes/pki/etcd/ca.crt"
ETCD_CERT="/etc/kubernetes/pki/etcd/server.crt"
ETCD_KEY="/etc/kubernetes/pki/etcd/server.key"
BACKUP_DIR="/backup/etcd"
DATE=$(date +%Y%m%d_%H%M%S)

etcdctl --endpoints=${ETCD_ENDPOINTS} \
  --cacert=${ETCD_CACERT} \
  --cert=${ETCD_CERT} \
  --key=${ETCD_KEY} \
  snapshot save ${BACKUP_DIR}/etcd-snapshot-${DATE}.db

# 保留最近7天备份
find ${BACKUP_DIR} -name "etcd-snapshot-*.db" -mtime +7 -delete
```

### 9.2 恢复流程

```bash
# 恢复etcd数据
etcdctl snapshot restore /backup/etcd/etcd-snapshot-xxx.db \
  --data-dir=/var/lib/etcd-restore

# 停止etcd，替换数据目录，重启服务
```

### 9.3 应用备份

- Harbor数据：定期备份 `/data/harbor` 目录
- Jenkins配置：备份 JENKINS_HOME
- Loki日志：配置保留策略，定期归档

## 10. 监控告警

### 10.1 告警规则示例

```yaml
# prometheus-rules.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: devops-alerts
  namespace: monitoring
spec:
  groups:
  - name: node-alerts
    rules:
    - alert: NodeHighCPU
      expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "节点CPU使用率过高"
        description: "节点 {{ $labels.instance }} CPU使用率超过80%"
    - alert: NodeHighMemory
      expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "节点内存使用率过高"
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[15m]) * 60 * 15 > 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Pod频繁重启"
```

### 10.2 告警通知

- 配置 Alertmanager
- 对接企业微信/钉钉/邮件通知

## 11. 运维建议

### 11.1 日常运维

- 每日检查集群健康状态
- 定期备份 etcd 数据
- 监控集群资源使用率
- 定期清理过期镜像（Harbor配置保留策略）
- 检查日志存储空间

### 11.2 故障恢复

- Master故障: 等待自动恢复或手动干预
- 节点故障: Pod自动迁移到其他节点
- 存储故障: 检查NFS服务状态
- 网络故障: 检查Calico/网络设备

## 12. 附录

### 12.1 参考文档

- Kubernetes官方文档: https://kubernetes.io/docs/
- Jenkins Kubernetes Plugin: https://plugins.jenkins.io/kubernetes/
- Harbor文档: https://goharbor.io/docs/
- Loki文档: https://grafana.com/docs/loki/
- Kaniko文档: https://github.com/GoogleContainerTools/kaniko
- Cas Protocol: https://apereo.github.io/cas/6.6.x/protocol/CAS-Protocol.html

### 12.2 待确认事项

- [ ] 实际服务器数量和配置
- [ ] 内网网段规划
- [ ] CAS系统具体版本和配置信息
- [ ] Gitee访问地址
- [ ] 是否需要多机房部署
- [ ] 告警通知渠道（企业微信/钉钉/邮件）
