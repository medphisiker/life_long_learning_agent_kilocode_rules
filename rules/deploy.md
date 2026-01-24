# Deployment Rules

В проекте поддерживаются два режима развертывания: **DEV** (разработка) и **PROD** (эксплуатация). Оба режима идентичны по структуре сетей, портам и используемым переменным окружения, что гарантирует консистентность работы системы.

## Основные отличия

| Характеристика | DEV (Разработка) | PROD (Эксплуатация) |
| :--- | :--- | :--- |
| **Скрипты запуска** | `start-dev.sh`, `stop-dev.sh` | `start-prod.sh`, `stop-prod.sh` |
| **Docker Compose** | `docker-compose-dev.yml` | `docker-compose-prod.yml` |
| **Образы (Images)** | Собираются локально (`build: .`) | Загружаются из GHCR (`image: ghcr.io/...`) |
| **Код (Source Code)** | Монтируется внутрь (`volumes: ./src:/app`) | Запечен внутри образа |
| **Перезагрузка** | Hot-reload (через монтирование) | Требует пересборки/перезапуска образа |

- **DEV**: Использование скриптов `start-dev.sh` и `stop-dev.sh` для запуска групп сервисов с иерархической группировкой. Подробнее: [`docs/deployment/scripts.md`](docs/deployment/scripts.md).
- **PROD**: Использование скриптов `start-prod.sh` и `stop-prod.sh`. Используются фиксированные образы из GHCR.
- **Bootstrap**: Для первичного развертывания на новом сервере используется скрипт `init-new-server.sh`. Он поддерживает аргументы `prod` (по умолчанию) и `dev`. Скрипт инициализирует базы данных, загружает знания из бэкапов, настраивает роли и выполняет массовое создание пользователей (если есть `users_to_create.json`). Подробнее: [`docs/deployment/update-prod.md`](docs/deployment/update-prod.md).
- **Network**: Все сервисы (DEV и PROD) используют внешние Docker-сети (`external: true`), которые создаются скриптами запуска. Это предотвращает конфликты меток при использовании нескольких проектов Compose.
- **Volumes**: RAG сервис использует внешние тома (`rag_qdrant_storage`, `rag_redis_data`), которые являются общими для DEV и PROD, что гарантирует сохранность данных.
- **Ports**: Порты в DEV и PROD идентичны, что обеспечивает прозрачность разработки.
- **Test**: After push to GHCR, always test PROD by deleting local images and running `docker-compose-prod.yml`.
- **Backup**: Используйте скрипт `backup-envs.sh` для создания резервной копии всех конфигурационных файлов `.env`.

## Infrastructure

### Сетевое взаимодействие (Aliases)
Для стабильного взаимодействия во всех режимах используются следующие алиасы внутри Docker сетей:
- `agent-service` — Оркестратор.
- `rag-api` — Поиск по базе знаний.
- `rag-qdrant` — Векторная БД Qdrant.
- `rag-redis` — Кэш Redis.
- `test-generator-api` — Генерация тестов.
- `user-service` — Управление пользователями.
- `web-backend` — Бэкенд интерфейса.
- `web-frontend` — Фронтенд интерфейса.

### Именование контейнеров
Для предотвращения конфликтов и удобства управления, имена контейнеров имеют суффиксы, соответствующие окружению:
- **DEV**: `*-dev` (например, `rag-api-dev`, `redis-dev`).
- **PROD**: `*-prod` (например, `rag-api-prod`, `redis-prod`).

### Ключевые порты (Внешние)
- `8150` — Web UI Frontend.
- `8151` — Web UI Backend.
- `8270` — Agent Service.
- `8000` — RAG API.
- `52812` — Test Generator API.
- `8010` — User Service API.

## PROD Images

Для работы PROD окружения необходимы следующие образы (тег `v001`):
- `ghcr.io/lifelong-learning-assisttant/rag-api:v001`
- `ghcr.io/lifelong-learning-assisttant/test_generator:v001`
- `ghcr.io/lifelong-learning-assisttant/web_ui_backend:v001`
- `ghcr.io/lifelong-learning-assisttant/web_ui_frontend:v001`
- `ghcr.io/lifelong-learning-assisttant/agent_service:v001`
- `ghcr.io/lifelong-learning-assisttant/user_service:v001`