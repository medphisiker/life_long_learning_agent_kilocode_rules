# Deployment Rules

- **DEV**: Использование скриптов `start-dev.sh` и `stop-dev.sh` для запуска групп сервисов с иерархической группировкой. Подробнее: [`docs/deployment/scripts.md`](docs/deployment/scripts.md).
- **PROD**: Использование скриптов `start-prod.sh` и `stop-prod.sh`. Используются фиксированные образы из GHCR.
- **Network**: PROD services must use external networks expected by `agent_service`.
- **Ports**: PROD ports must strictly match `agent_service` environment variables.
- **Test**: After push to GHCR, always test PROD by deleting local images and running `docker-compose-prod.yml`.

## PROD Infrastructure

### Сетевое взаимодействие (Aliases)
Для стабильного взаимодействия в PROD используются следующие алиасы внутри Docker сетей:
- `agent-service` — Оркестратор.
- `rag-api` — Поиск по базе знаний.
- `test-generator-api` — Генерация тестов.
- `web-backend` — Бэкенд интерфейса.
- `web-frontend` — Фронтенд интерфейса.

### Ключевые порты (Внешние)
- `8150` — Web UI Frontend.
- `8151` — Web UI Backend.
- `8270` — Agent Service.
- `8000` — RAG API.
- `52812` — Test Generator API.

## PROD Images

Для работы PROD окружения необходимы следующие образы (тег `v001`):
- `ghcr.io/lifelong-learning-assisttant/rag-api:v001`
- `ghcr.io/lifelong-learning-assisttant/test_generator:v001`
- `ghcr.io/lifelong-learning-assisttant/web_ui_backend:v001`
- `ghcr.io/lifelong-learning-assisttant/web_ui_frontend:v001`
- `ghcr.io/lifelong-learning-assisttant/agent_service:v001`