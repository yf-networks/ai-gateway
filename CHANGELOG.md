# Changelog

## [v0.1.1] — 2026-07-15

### Updated Components

| Component | Version |
|---|---|
| BFE | v1.8.4 (develop build) |
| AI Gateway API | v0.0.4 (develop build) |
| Dashboard | [v0.0.4](https://github.com/yf-networks/ai-gateway-web/releases/tag/v0.0.4) |
| conf-agent | v0.0.4 ([yf-networks](https://github.com/yf-networks/conf-agent)) |

### Changed

- Bump BFE to v1.8.4 (fix token calculation, allow_models check, session_sticky redis unmarshal).
- Bump AI Gateway API to v0.0.4 (mod_body_process config export, PATCH/PUT Entity, allow_models intersection optimization).
- Bump Dashboard to v0.0.4 (certificate management, grouped model selectors, i18n improvements).

### Added

- MySQL persistent storage guide for K8s (`deploy/mysql-pvc.yaml`), with init Job skipping already-initialized databases.

### Fixed

- Fix `Exec format error` on amd64 servers when image is built on Apple Silicon (https://github.com/yf-networks/ai-gateway/issues/1).
- Fix `config.InitLog — logger exception must be set` by adding missing `[Loggers.exception]` to config template.
- Fix BFE startup failure due to missing `QuotaPlans` in default `token_rule.data`.
- Fix entrypoint to dump process logs to `docker logs` on startup failure.
- Fix `docker compose` prerequisite instructions for Linux users.

---

## [v0.1.0] — 2026-07-10

First official release of AI Gateway — a product-level entry point that unifies version management across all sub-components. Replaces `ai-gateway-demo`, adding all-in-one container deployment alongside existing K8s manifests.

### Component Versions

| Component | Version |
|---|---|
| BFE | [v1.8.3](https://github.com/bfenetworks/bfe/releases/tag/v1.8.3) |
| AI Gateway API | [v0.0.3](https://github.com/yf-networks/ai-gateway-api/releases/tag/v0.0.3) |
| Dashboard | v0.0.3 ([ai-gateway-web](https://github.com/yf-networks/ai-gateway-web)) |
| conf-agent | v0.0.3 ([yf-networks](https://github.com/yf-networks/conf-agent)) |
| Service Controller | v0.0.1 |

### Added

- All-in-One container deployment via `Dockerfile.standalone` + `docker-compose.yml`.
- One-command startup: `docker compose up -d` launches MySQL + Redis + AI Gateway.
- Automatic database initialization on first boot (`api_db_ddl.sql` with full seed data).
- Docker network DNS using the same naming convention as K8s Service DNS.
- Test simulator via `docker compose --profile test up -d`.
- Multi-arch images: `linux/amd64` and `linux/arm64`.
- `VERSIONS.yaml` as single source of truth for component versions.
- Kubernetes deployment manifests migrated from `ai-gateway-demo`.
- Bilingual README (EN/CN), BUILD_GUIDE, K8s documentation, CONTRIBUTING.
