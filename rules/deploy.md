# Deployment Rules

- **DEV**: Использование скриптов `start-dev.sh` и `stop-dev.sh` для запуска групп сервисов с иерархической группировкой. Подробнее: [`docs/deployment_scripts.md`](docs/deployment_scripts.md).
- **PROD**: Корневой `docker-compose-prod.yml` с фиксированными образами из GHCR.
- **Network**: PROD services must use external networks expected by `agent_service`.
- **Ports**: PROD ports must strictly match `agent_service` environment variables.
- **Test**: After push to GHCR, always test PROD by deleting local images and running `docker-compose-prod.yml`.