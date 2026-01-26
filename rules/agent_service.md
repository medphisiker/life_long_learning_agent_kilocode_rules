# Сервис Агента (agent_service)

Основной оркестратор системы, реализующий логику LLM-агента, управление состоянием сессий и интеграцию с внешними инструментами (RAG, генератор тестов).

### Многопользовательская работа и конкурентность (v3.0)
- **Изоляция**: Каждая сессия пользователя (`session_id`) изолирована. Состояние хранится в `AgentSession` и в чекпоинтах LangGraph (`MemorySaver`) с привязкой к `thread_id`.
- **Конкурентность**: Лимит одновременных генераций управляется через `concurrency_limit` (по умолчанию **5**). При превышении запросы становятся в очередь.
- **Жизненный цикл**: Сессии имеют TTL (по умолчанию 600с). Фоновый процесс (Sweeper) очищает неактивные сессии.
- **Подробности**: См. [`agent_service/docs/concurrency_and_sessions.md`](agent_service/docs/concurrency_and_sessions.md).

### Динамические настройки (v2.0)
Сервис поддерживает динамическую смену провайдеров и моделей LLM для каждого запроса через объект `AppSettings`. Настройки пробрасываются в инструменты (RAG, Test Generator) для консистентности ответов.

## Ключевые файлы (Key Files Map)

- [`app.py`](agent_service/app.py:1) — **Точка входа FastAPI**. Настройка роутов, middleware и WebSocket-интеграции.
- [`agent_system.py`](agent_service/agent_system.py:1) — **Оркестратор сессий**. Управляет пулом `AgentSession`, лимитами конкурентности и очисткой ресурсов.
- [`agent_session.py`](agent_service/agent_session.py:1) — **Ядро агента (State Machine)**. Реализация графа LangGraph, управление состоянием отдельного диалога и уведомлениями UI.
- [`llm_service/llm_client.py`](agent_service/llm_service/llm_client.py:1) — **Универсальный LLM клиент**. Абстракция над провайдерами (OpenRouter, Z.ai и др.).
- [`prompts/`](agent_service/prompts/:1) — **Библиотека промптов**. Системные инструкции, шаблоны классификации интентов и подготовки материалов.
- [`langchain_tools.py`](agent_service/langchain_tools.py:1) — **Инструменты агента**. Определение функций, которые LLM может вызывать (RAG search, Test generation).
- [`graphs/`](agent_service/graphs/:1) — **Подграфы LangGraph**:
    - `chat.py` — Логика свободного общения (Router, Direct Answer, Full Answer).
    - `retrieval.py` — Логика поиска (RAG, Web, Docs).
- [`settings.py`](agent_service/settings.py:1) — Глобальная конфигурация через Pydantic Settings.
- `tests/` — Тестовое покрытие:
    - `components/` — Юнит-тесты компонентов и интеграций с провайдерами.
    - `pipeline/` — Интеграционные тесты сценариев (RAG, Quiz, Chit-chat).
    - `scenarios/` — Тесты отдельных подграфов (Chat, Retrieval).

## Файлы конфигурации

- `docker-compose-dev.yml` — Сборка для разработки с пробросом кода (volumes). Использует проект `lifelong_learning-agent`.
- `docker-compose-prod.yml` — Запуск стабильной версии из GHCR с фиксированным образом (`ghcr.io/lifelong-learning-assisttant/agent_service:v001`).
- `app_settings-dev.json` / `app_settings-prod.json` — Настройки уровней окружения. В DEV используются алиасы (например, `http://agent-service:8270`).
- `.env_example` — Шаблон переменных окружения (API ключи).

## Ключевые порты
- `8250` — API сервис агента (DEV).
- `8270` — API сервис агента (PROD).

## Сетевое взаимодействие (Docker Networks)

Сервис агента взаимодействует с другими компонентами системы через выделенные внешние Docker-сети. Для стабильности взаимодействия во всех режимах используются **внешние сети** (`external: true`) и **сетевые алиасы**, что позволяет избежать проблем с динамическими именами контейнеров Docker Compose.

### Сетевые алиасы
- `rag-api`, `rag-qdrant`, `rag-redis` — компоненты RAG.
- `test-generator-api` — генератор тестов.
- `agent-service` — сам агент.
- `web-backend`, `web-frontend` — интерфейс.

### Docker Networks
- `rag_rag_network` — для связи с RAG.
- `test_generator_default` — для связи с генератором тестов.
- `web_ui_network` — для связи с Web UI.

При запуске через `docker-compose-dev.yml` агент ожидает, что сервисы `rag`, `test_generator` и `web_ui_service` уже запущены и их сети доступны.

### Важное при разработке (v3.1)
- **Перезапуск**: Для корректного применения изменений в коде и зависимостях рекомендуется использовать глобальные скрипты в корне проекта:
    ```bash
    ./stop-dev.sh
    ./start-dev.sh
    ```
- **Изоляция**: Прямой запуск через `docker compose` внутри папки сервиса может привести к конфликтам имен контейнеров или отсутствию необходимых внешних сетей.

## Поддерживаемые провайдеры LLM
- **OpenAI** (через `ai-mediator.ru`, модель по умолчанию `gemini-3-flash-preview`)
- **Z.ai** (модель `glm-4.6v`)
- **OpenRouter**
- **Mistral**

## NetRunner Protocol (v3.2)
- Спецификация вынесена в [`netrunner_protocol.md`](netrunner_protocol.md).