[English](README.md) | 简体中文

# AI Gateway Kubernetes 部署

## 架构图

![Kubernetes](../.images/ai-gateway-k8s.png)

本部署在 `ai-gateway-system` 命名空间中演示了各关键组件的交互：

- **数据面**（bfe + conf-agent）：流量转发与接入控制
- **控制面**（ai-gateway-api）：策略/配置下发接口
- **基础依赖**（MySQL、Redis）：为控制面提供存储与依赖服务
- **服务发现**（service-controller）：发现并同步后端服务
- **示例后端**（llm-d inference simulator）：验证路由

组件间通过 Kubernetes Service/DNS 通信：
- `ai-gateway-api.ai-gateway-system.svc.cluster.local`
- `mysql.ai-gateway-system.svc.cluster.local`
- `redis.ai-gateway-system.svc.cluster.local`

> 注意：MySQL / Redis 使用 `emptyDir` 存储，Pod 重启后数据丢失。仅用于演示和联通性验证。

## 清单概览

| 文件 | 说明 |
|---|---|
| `namespace.yaml` | 命名空间定义（ai-gateway-system） |
| `kustomization.yaml` | Kustomize 资源汇总与镜像覆盖 |
| `bfe-configmap.yaml` | BFE 配置（bfe.conf、conf-agent.toml） |
| `bfe-deploy.yaml` | BFE 数据面 Deployment |
| `ai-gateway-configmap.yaml` | AI Gateway API 配置（DB/Redis、鉴权） |
| `ai-gateway-deploy.yaml` | AI Gateway API Deployment 与 Service |
| `mysql-deploy.yaml` | MySQL（Deployment、Service、初始化 ConfigMap、初始化 Job） |
| `redis-deploy.yaml` | Redis Deployment 与 Service |
| `service-controller-deploy.yaml` | 服务发现控制器 |
| `llm-d-inference-sim-deploy.yaml` | 示例后端推理模拟服务 |

## 快速开始

### 前置条件

- kubectl >= 1.20，支持 `-k`
- 集群管理权限（Namespace、Deployment、Service、ConfigMap、Secret）
- 集群节点可拉取镜像

### 1. 配置镜像（可选）

如需使用自定义镜像地址或版本，修改 `kustomization.yaml` 中的 `images:` 配置：

```yaml
images:
  - name: ghcr.io/bfenetworks/bfe
    newName: ghcr.io/your-org/bfe
    newTag: v1.8.2
  - name: ghcr.io/yf-networks/ai-gateway-api
    newName: ghcr.io/your-org/ai-gateway-api
    newTag: v0.0.2
  - name: ghcr.io/bfenetworks/service-controller
    newName: ghcr.io/your-org/service-controller
    newTag: v0.0.1
```

### 2. 一键部署

```bash
kubectl apply -k .
```

部署：bfe（含 conf-agent）、ai-gateway-api（含 Dashboard）、mysql、redis、service-controller。

### 3. 部署测试服务（可选）

```bash
kubectl apply -f deploy/llm-d-inference-sim-deploy.yaml
```

> 部署在 `default` 命名空间。如需替换镜像/模型参数，直接编辑该文件。

### 4. 验证

```bash
kubectl get pods -n ai-gateway-system
kubectl get svc -n ai-gateway-system
```

访问 Dashboard：`http://{NodeIP}:30183`（admin / admin）

## 使用外部数据库

演示用的 MySQL 使用 `emptyDir` 存储，生产环境建议：

1. 在外部 MySQL 实例上执行 `db_ddl.sql`
2. 修改 `ai-gateway-configmap.yaml` 中的数据库连接信息
3. 在 `kustomization.yaml` 中注释掉 `mysql-deploy.yaml`

## 后端服务要求

Service Controller 通过监听 Kubernetes Service 的标签来发现后端服务：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: <service-name>
  labels:
    bfe-product: AI_product  # 必须：固定值
spec:
  ports:
    - name: http             # 必须：端口必须命名
      port: 8080
      targetPort: 80
```

- `bfe-product`：必须为 `AI_product`
- `spec.ports[].name`：必填，可为任意有意义的名称（http、https、grpc 等）

## 清理

```bash
kubectl delete -f llm-d-inference-sim-deploy.yaml
kubectl delete -k .
```

> 建议先删除示例后端再删 `ai-gateway-system`，避免 finalizers 导致卡住。

## 故障排查

### 镜像拉取失败

```bash
kubectl describe pod -n ai-gateway-system <pod-name>
```

常见原因：`kustomization.yaml` 中镜像配置错误，缺少 `imagePullSecrets`。

### 控制面 CrashLoopBackOff

```bash
kubectl logs -n ai-gateway-system -l app=ai-gateway-api --tail=200
```

常见原因：MySQL/Redis 未就绪，连接配置错误，未执行数据库初始化脚本。

### BFE 返回 500

未配置转发规则时该返回值符合预期。在 Dashboard 中配置规则后验证：

```bash
curl -v http://{NodeIP}:30080/
```
