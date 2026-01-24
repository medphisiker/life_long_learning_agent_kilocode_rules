# Сервис генерации тестов (test_generator)

Сервис на базе FastAPI для автоматической генерации вопросов, проведения экзаменов и оценки ответов с использованием LLM (OpenAI/Yandex).

## Ключевые файлы (Key Files Map)

- [`app/api/`](test_generator/app/api/:1) — Эндпоинты FastAPI (генерация, проверка ответов, загрузка файлов, healthcheck).
- [`app/core/generator.py`](test_generator/app/core/generator.py:1) — **Логика создания вопросов**.
- [`app/core/grader.py`](test_generator/app/core/grader.py:1) — **Оценка ответов пользователя**.
- [`app/core/parser.py`](test_generator/app/core/parser.py:1) — **Парсинг входных данных** (Markdown).
- [`app/core/exam_builder.py`](test_generator/app/core/exam_builder.py:1) — Сборка структуры экзамена.
- [`app/prompts/`](test_generator/app/prompts/:1) — Системные промпты для LLM (RU/EN).
- `data/` — Локальное хранилище данных (загрузки, результаты).
- `docs/` — Техническая документация и архитектурные планы.
- `tests/` — Тестовое покрытие (Unit, Integration, BDD). Команды запуска в [`testing.md`](testing.md).
- `examples/` — Примеры использования и примеры входных данных.

## Файлы конфигурации

- `docker-compose-dev.yml` — Локальная сборка и запуск для разработки.
- `docker-compose-prod.yml` — Запуск стабильной версии из GHCR.
- `.env` — Переменные окружения (API ключи, настройки моделей).
- `Dockerfile` — Инструкции для сборки образа.

## Ключевые порты
- `52812` (внешний в PROD) -> `8000` (внутренний в контейнере).