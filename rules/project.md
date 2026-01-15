# Проект

Наш проект это образовательный сервис с LLM-агентом, который помогает пользователю в подготовке к собеседованиям по ML/DL и алгоритмам в big tech it.

Проект модульный и состоит из отдельный компонентов системы:

- Сервис Агента — основной оркестратор системы (`./agent_service`). Подробнее: [`.kilocode/rules/agent_service.md`](.kilocode/rules/agent_service.md)
- RAG сервис реализующий гибридный и смантический поиск по учебнику Яндекса по ML (`./rag`). Подробнее: [`.kilocode/rules/rag.md`](.kilocode/rules/rag.md)
- Генератор квизов, позволяющий генерировать вопросы на основе markdown файлов (`./test_generator`). Подробнее: [`.kilocode/rules/test_generator.md`](.kilocode/rules/test_generator.md)
- Web UI веб интерфейс с котьорым взаимодествует ползователь (`./web_ui_service`). Подробнее: [`.kilocode/rules/web_ui_service.md`](.kilocode/rules/web_ui_service.md)