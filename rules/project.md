# Проект

Наш проект это образовательный сервис с LLM-агентом, который помогает пользователю в подготовке к собеседованиям по ML/DL и алгоритмам в big tech it.

Проект модульный и состоит из отдельных компонентов системы:

- Сервис Агента — основной оркестратор системы (`./agent_service`). Подробнее: [`agent_service/README.md`](agent_service/README.md) и [`.kilocode/rules/agent_service.md`](.kilocode/rules/agent_service.md)
- RAG сервис реализующий гибридный и семантический поиск по учебнику Яндекса по ML (`./rag`). Подробнее: [`rag/README.md`](rag/README.md) и [`.kilocode/rules/rag.md`](.kilocode/rules/rag.md)
- Генератор квизов, позволяющий генерировать вопросы на основе markdown файлов (`./test_generator`). Подробнее: [`test_generator/README.md`](test_generator/README.md) и [`.kilocode/rules/test_generator.md`](.kilocode/rules/test_generator.md)
- Web UI веб интерфейс с которым взаимодействует пользователь (`./web_ui_service`). Подробнее: [`web_ui_service/README.md`](web_ui_service/README.md) и [`.kilocode/rules/web_ui_service.md`](.kilocode/rules/web_ui_service.md)

## Тестирование

Все тесты в проекте должны запускаться внутри соответствующих Docker-контейнеров. Это гарантирует наличие всех зависимостей, правильных путей и доступа к необходимым переменным окружения (API ключи и т.д.).

**Общий шаблон запуска:**
```bash
docker exec <container_name> <test_command>
```

**Примеры для DEV окружения:**
- **Агент**: `docker exec lifelong_learning-agent-agent_dev-1 env APP_SETTINGS_PATH=app_settings-dev.json uv run pytest tests/`
- **RAG**: `docker exec rag-api-dev uv run pytest tests/`
- **Генератор тестов**: `docker exec llm-tester-api-dev uv run pytest tests/`
- **NetRunner Scenarios**: `docker exec web_ui_service-backend-dev uv run python /app/tests_integration/test_netrunner_scenarios.py --cfg /app/tests_integration/config-dev.json`