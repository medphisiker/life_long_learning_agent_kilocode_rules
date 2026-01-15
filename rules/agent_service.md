# Сервис Агента (agent_service)

Основной оркестратор системы, реализующий логику LLM-агента, управление сессиями и интеграцию с внешними инструментами (RAG, генератор тестов, Tavily, Context7).

## Структура проекта

- `app.py` — Точка входа FastAPI, настройка роутов и middleware.
- `agent_system.py` — Глобальная система управления агентами и сессиями.
- `agent_session.py` — Логика отдельной пользовательской сессии (state machine агента).
- `llm_service/` — Модуль взаимодействия с LLM провайдерами.
    - `llm_client.py` — Универсальный клиент (OpenAI, OpenRouter, Mistral, Z.ai).
    - `utils.py` — Вспомогательные функции для обработки HTTP-ответов и таймаутов.
- `prompts/` — Шаблоны системных и функциональных промптов.
- `langchain_tools.py` — Определение инструментов (tools) для LangChain агента.
- `settings.py` — Конфигурация через Pydantic Settings.
- `prompt_loader.py` — Загрузчик текстовых промптов из файлов.
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
- `8270` — API сервис агента (единый порт для DEV и PROD).

## Сетевое взаимодействие (Docker Networks)

Сервис агента взаимодействует с другими компонентами системы через выделенные внешние Docker-сети. Для стабильности взаимодействия в DEV режиме используются **сетевые алиасы**, что позволяет избежать проблем с динамическими именами контейнеров Docker Compose.

### Сетевые алиасы (DEV)
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