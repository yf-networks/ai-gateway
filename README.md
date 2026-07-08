[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Go](https://img.shields.io/badge/Go-1.22+-00ADD8.svg)](https://go.dev/)

[English](README.md) | [简体中文](README_CN.md)

# AI Gateway

AI Gateway is an open-source AI traffic gateway built on top of [BFE](https://github.com/bfenetworks/bfe). It provides unified API management, authentication, rate limiting, and intelligent routing for multiple AI model providers, giving developers a single entry point for all AI services.

## Architecture

![Architecture](./.images/deploy_architecture_ai.png)

AI Gateway consists of the following core components:

| Component | Role | Description | Repository |
|---|---|---|---|
| **AI Gateway API** | Control plane | Open APIs for policy/config management and distribution | [yf-networks/ai-gateway-api](https://github.com/yf-networks/ai-gateway-api) |
| **Dashboard** | Admin console | Web UI for visual management (bundled in API image) | [yf-networks/ai-gateway-web](https://github.com/yf-networks/ai-gateway-web) |
| **BFE** | Data plane | Traffic forwarding and access control | [bfenetworks/bfe](https://github.com/bfenetworks/bfe) |
| **Conf Agent** | Config agent | Fetches config and triggers BFE hot reload | [bfenetworks/conf-agent](https://github.com/bfenetworks/conf-agent) |
| **Service Controller** | Service discovery | Discovers and syncs K8s backend services (K8s only) | [bfenetworks/service-controller](https://github.com/bfenetworks/service-controller) |

## Key Features

- **AI route management**: Route configuration for multiple AI model providers (OpenAI, DeepSeek, Anthropic, Google Gemini, etc.)
- **API key management**: Create, delete, and validate API keys for AI services
- **Domain management**: Bind domains and configure routing rules
- **Certificate management**: Upload and manage TLS certificates
- **Cluster/sub-cluster management**: Manage backend service clusters
- **Traffic management**: Traffic allocation and scheduling
- **Dashboard**: Web-based management console (bundled in API image)
- **Config export**: Export configuration for BFE data plane and Conf Agent

## Deployment Modes

AI Gateway supports two deployment modes:

| Mode | Command | Use Case |
|---|---|---|
| **Container** | `docker run` | Development, demo, small-scale deployment |
| **Kubernetes** | `kubectl apply -k kubernetes/` | Production, cluster deployment |

### Container (All-in-One Docker)

Single container that bundles BFE + AI Gateway API + Conf Agent, with automatic database initialization.

**Prerequisites**: Docker, MySQL 8, Redis 6.2

```bash
# Pull image
docker pull ghcr.io/yf-networks/ai-gateway:latest

# Prepare config files (see conf/ for templates)
# Edit conf/ai_gateway_api.toml with your DB credentials

# Start
docker run -d --name ai-gateway \
  -p 8080:8080 -p 8183:8183 \
  -v $(pwd)/conf/ai_gateway_api.toml:/home/work/api-server/conf/ai_gateway_api.toml \
  -v $(pwd)/conf/name_conf.data:/home/work/api-server/conf/name_conf.data \
  -v $(pwd)/conf/name_conf.data:/home/work/bfe/conf/name_conf.data \
  -v $(pwd)/conf/bfe.conf:/home/work/bfe/conf/bfe.conf \
  ghcr.io/yf-networks/ai-gateway:latest
```

Access Dashboard: `http://localhost:8183` (admin / admin)

### Kubernetes

**Prerequisites**: kubectl >= 1.20, cluster admin permissions

```bash
# Deploy all components with one command
kubectl apply -k kubernetes/

# Verify
kubectl get pods -n ai-gateway-system
```

Access Dashboard: `http://{NodeIP}:30183` (admin / admin)

For complete deployment guide and architecture diagram, see the [Kubernetes documentation](./kubernetes/README.md).

## Quick Start

### 1. Clone

```bash
git clone https://github.com/yf-networks/ai-gateway.git
cd ai-gateway
```

### 2. Configure

Copy and edit the config files in `conf/`:

| File | Purpose |
|---|---|
| `conf/ai_gateway_api.toml` | Database, Redis, and API server settings |
| `conf/bfe.conf` | BFE traffic gateway configuration |
| `conf/name_conf.data` | Redis instance discovery (shared by API and BFE) |

Example `conf/ai_gateway_api.toml` (minimal changes):

```toml
[Databases.bfe_db]
Addr   = "192.168.1.3:3306"      # MySQL address
User   = "root"                  # MySQL user
Passwd = "your_password"         # MySQL password

[RedisConf]
Bns = "BFE.poc-redis-wx"         # Redis BNS name
```

Example `conf/name_conf.data` (Redis instance):

```json
{
    "Version": "init version",
    "Config": {
        "BFE.poc-redis-wx": [{
            "Host": "192.168.1.4",
            "Port": 6379,
            "Weight": 10
        }]
    }
}
```

### 3. Choose deployment mode

- **Container**: See [Container section](#container-all-in-one-docker) above
- **Kubernetes**: See [Kubernetes section](#kubernetes) above

## Build from Source

To build the all-in-one image from source components:

```bash
# Build container image
make docker-standalone

# Build with debug tools (curl, vim, etc.)
make docker-standalone VARIANT=debug

# Multi-arch push
make docker-standalone-push REGISTRY=ghcr.io/your-org
```

**Build parameters**:

| Parameter | Default | Description |
|---|---|---|
| `BFE_IMAGE` | `ghcr.io/bfenetworks/bfe:v1.8.2` | BFE image (provides bfe + conf-agent) |
| `API_IMAGE` | `ghcr.io/yf-networks/ai-gateway-api:v0.0.2` | API server image (provides api-server + dashboard) |
| `VARIANT` | `prod` | `prod` or `debug` |

## Version Management

This repository serves as the **product-level version entry point** for AI Gateway. The `VERSIONS.yaml` file defines the compatibility matrix between all sub-components:

```yaml
version: 0.2.0
components:
  bfe:
    version: v1.8.2
    source: image
    image: ghcr.io/bfenetworks/bfe:v1.8.2
    provides:
      - bfe
      - conf-agent
  ai-gateway-api:
    version: v0.0.2
    source: image
    image: ghcr.io/yf-networks/ai-gateway-api:v0.0.2
```

Each product release corresponds to a verified combination of component versions. See `VERSIONS.yaml` for the full matrix.

## Exposed Ports

| Port | Component | Purpose |
|---|---|---|
| 8080 | BFE | HTTP entry |
| 8443 | BFE | HTTPS entry |
| 8421 | BFE | Monitor |
| 8183 | API Server | API + Dashboard |
| 8284 | API Server | Monitor |

## Contributing

Contributions are welcome! See [CONTRIBUTING.md](./CONTRIBUTING.md) for the development workflow and guidelines.

## License

AI Gateway is released under the [Apache License 2.0](LICENSE).

## References

- [BFE](https://github.com/bfenetworks/bfe) — Data plane engine
- [AI Gateway API](https://github.com/yf-networks/ai-gateway-api) — Control plane
- [AI Gateway Web](https://github.com/yf-networks/ai-gateway-web) — Dashboard frontend
- [Conf Agent](https://github.com/bfenetworks/conf-agent) — Configuration agent
- [Service Controller](https://github.com/bfenetworks/service-controller) — K8s service discovery
