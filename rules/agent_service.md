# Сервис Агента (agent_service)

Основной оркестратор системы, реализующий логику LLM-агента, управление состоянием сессий и интеграцию с внешними инструментами (RAG, генератор тестов).

### Многопользовательская работа и конкурентность (v3.0)
- **Изоляция**: Каждая сессия пользователя (`session_id`) изолирована. Состояние хранится в `AgentSession` и в чекпоинтах LangGraph (`MemorySaver`) с привязкой к `thread_id`.
- **Конкурентность**: Лимит одновременных генераций управляется через `concurrency_limit` (по умолчанию **5**). При превышении запросы становятся в очередь.
- **Жизненный цикл**: Сессии имеют TTL (по умолчанию 600с). Фоновый процесс (Sweeper) очищает неактивные сессии.
- **Подробности**: См. [`agent_service/docs/concurrency_and_sessions.md`](agent_service/docs/concurrency_and_sessions.md).

### Динамические настройки (v2.0)
Сервис поддерживает динамическую смену провайдеров и моделей LLM для каждого запроса через объект `AppSettings`. Настройки пробрасываются в инструменты (RAG, Test Generator) для консистентности ответов.

## Структура проекта

- `app.py` — Точка входа FastAPI, настройка роутов и middleware.
- `agent_system.py` — Глобальная система управления агентами и состоянием сессий.
- `agent_session.py` — Логика отдельной сессии агента (state machine).
- `llm_service/` — Модуль взаимодействия с LLM провайдерами.
    - `llm_client.py` — Универсальный клиент (OpenAI, OpenRouter, Mistral, Z.ai).
    - `utils.py` — Вспомогательные функции для обработки HTTP-ответов и таймаутов.
- `prompts/` — Шаблоны системных и функциональных промптов.
- `langchain_tools.py` — Определение инструментов (tools) для LangChain агента.
- `settings.py` — Конфигурация через Pydantic Settings.
- `logger.py` — Настройка логирования.
- `tests/` — Тестовое покрытие:
    - `components/` — Юнит-тесты компонентов и интеграций с провайдерами.
    - `pipeline/` — Интеграционные тесты сценариев (RAG, Quiz, Chit-chat).

## Файлы конфигурации

- `docker-compose-dev.yml` — Сборка для разработки с пробросом кода (volumes). Использует проект `lifelong_learning-agent`.
- `docker-compose-prod.yml` — Запуск стабильной версии из GHCR с фиксированным образом (`ghcr.io/anton-admin/agent_service:latest`).
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

## Поддерживаемые провайдеры LLM
- **Z.ai** (по умолчанию, модель `glm-4.6v`)
- **OpenRouter**
- **OpenAI**
- **Mistral**

## NetRunner Protocol (v3.2)
- **CLI-Priority**: Слэш-команды (`/finish_quizz`, `/skip_question`) обрабатываются в `planner_node` до вызова LLM.
- **Quiz Context**: При RAG-уточнениях в квизе обязательно возвращать пользователя к вопросу через маркер `[SYSTEM: QUIZ_CONTEXT_RESUMED]`.
- **Interaction Modes**: Поддержка `interaction_mode` (`AI_SYNC`, `ANSWER_QUIZ`). Режим `ANSWER_QUIZ` принудительно устанавливает интент `quiz_answering`.
- **Intents**: Новые типы `skip_question` и `evaluate_quiz` (для текстовых запросов).
- **Message Types**: Новый тип `quizz_question` для отделения вопросов теста от обычных сообщений ассистента.
- **Planner Logic (v3.2)**: В режиме активного квиза короткие сообщения без знака вопроса принудительно классифицируются как `quiz_answering`, даже если LLM определила `general` или `rag_answer`. Для корректной работы `interaction_mode` и `mode` пробрасываются в `AgentState`.