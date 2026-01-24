# RAG Сервис (rag)

Сервис для реализации гибридного и семантического поиска по учебнику Яндекса по ML. Использует Qdrant в качестве векторной БД и Redis для кэширования.

## Ключевые файлы (Key Files Map)

- [`app/main.py`](rag/app/main.py:1) — **Точка входа**. FastAPI роуты для `/search` и `/rag`.
- [`app/rag_service.py`](rag/app/rag_service.py:1) — **Логика RAG**. Координация между поиском (retrieval) и генерацией ответа LLM.
- [`app/retriever.py`](rag/app/retriever.py:1) — **Retriever**. Реализация гибридного поиска в Qdrant и кэширования в Redis.
- [`llm_service/llm_client.py`](rag/llm_service/llm_client.py:1) — Клиент для взаимодействия с моделями эмбеддингов и генерации.
    
    ## API Эндпоинты
    
    - `GET /health` — Проверка состояния сервиса. Возвращает статус подключения к Qdrant, Redis, наличие коллекции и **количество parent-документов в Redis** (`redis_parent_docs_count`).
    - `POST /search` — Поиск релевантных документов (гибридный поиск).
    - `POST /rag` — Генерация ответа на основе найденного контекста.
- `llm_service/` — Интеграция с языковыми моделями.
- `backup_downloader/` — Скрипты для скачивания бэкапов данных/индексов.
- `notebook/` — Jupyter ноутбуки для экспериментов (ETL, оценка метрик, Ragas).
- `tests/` — Наборы данных и скрипты для оценки качества RAG. Команды запуска в [`testing.md`](testing.md).
- `docs/` — Документация по миграции Qdrant и совместимости версий.

## Файлы конфигурации

- `docker-compose-dev.yml` — Окружение для разработки (локальная сборка). **Важно**: требует наличия внешних томов `rag_qdrant_storage` и `rag_redis_data`.
- `docker-compose-prod.yml` — Продакшн окружение (образы из GHCR).
- `docker-compose-bootstrap.yml` — Инициализация базы знаний. Использует скрипт `bootstrap_and_restore.sh` для скачивания бэкапов, очистки старых AOF файлов Redis и восстановления данных.
- `settings.py` — Глобальные настройки проекта.
- `.env.example` — Шаблон переменных окружения.

## Ключевые порты
- `8000` — API сервис.
- `6333` / `6334` — Qdrant (HTTP/gRPC).
- `6379` — Redis.
- `8081` — Redis Commander (Web UI для Redis).