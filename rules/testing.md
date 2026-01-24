# Правила тестирования (Testing Rules)

Этот файл содержит **единый источник правды** для запуска всех типов тестов в проекте.

**ВАЖНО:** Все тесты должны запускаться внутри Docker-контейнеров для обеспечения консистентности окружения.

## 1. Unit Tests (Модульные тесты)

Запускают проверку изолированной логики сервисов.

| Сервис | Команда запуска | Описание |
| :--- | :--- | :--- |
| **Agent Service** | `docker exec agent-service-dev env APP_SETTINGS_PATH=app_settings-dev.json uv run pytest tests/ -v` | Все тесты агента |
| **Agent (Sessions)** | `docker exec agent-service-dev uv run pytest tests/test_agent_session_updated.py tests/test_agent_system_sessions.py -v` | Тесты новой системы сессий |
| **Agent (Pipelines)**| `docker exec agent-service-dev uv run pytest tests/pipeline/ -v` | Тесты сценариев (RAG, Quiz) |
| **RAG Service** | `docker exec rag-api-dev uv run pytest tests/` | Тесты поиска и генерации |
| **Test Generator** | `docker exec test-generator-dev uv run pytest tests/` | Тесты генерации вопросов |
| **Web UI (Sim)** | `docker exec web_ui_service-backend-dev uv run pytest tests/test_web_ui_simulation.py -v` | Симуляция WebSocket без агента |

## 2. Integration Tests (NetRunner Scenarios)

Проверяют взаимодействие всей системы (Agent + RAG + Web UI + TestGen). Запускаются из контейнера `web_ui_service-backend-dev`.

**Конфиг:** `/app/tests_integration/config-dev.json`

| Сценарий | Команда запуска | Описание |
| :--- | :--- | :--- |
| **General Chat** | `docker exec web_ui_service-backend-dev uv run python /app/tests_integration/test_scenario_general.py --cfg /app/tests_integration/config-dev.json` | Простой диалог (General) |
| **RAG Search** | `docker exec web_ui_service-backend-dev uv run python /app/tests_integration/test_scenario_rag.py --cfg /app/tests_integration/config-dev.json` | Поиск по базе знаний |
| **Quiz Flow** | `docker exec web_ui_service-backend-dev uv run python /app/tests_integration/test_scenario_quiz.py --cfg /app/tests_integration/config-dev.json` | Полный цикл квиза |
| **Full User Flow** | `docker exec web_ui_service-backend-dev uv run python /app/tests_integration/test_user_flow.py` | Эмуляция действий пользователя |

## 3. Frontend & E2E

Тесты пользовательского интерфейса.

| Тип | Команда | Примечание |
| :--- | :--- | :--- |
| **React Tests** | *TBD* (см. `web_ui_service/frontend/package.json`) | Запуск через `npm test` внутри контейнера |
| **E2E (Playwright)**| *TBD* | Планируется |

## 4. RAG Evaluation

Оценка качества поиска и генерации.

| Задача | Команда |
| :--- | :--- |
| **Ragas Eval** | `docker exec rag-api-dev uv run python tests/evaluate_rag.py` |

## 5. Debugging Tips

- **Логи тестов**: Интеграционные тесты сохраняют логи в `web_ui_service/tests/logs/`.
- **WebSocket**: Для отладки WS используйте `test_web_ui_simulation.py`.
- **Session ID**: В интеграционных тестах ID сессии генерируется автоматически (например, `rag_test_<timestamp>`).