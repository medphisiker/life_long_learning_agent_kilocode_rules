# Web UI Сервис (web_ui_service)

Пользовательский интерфейс системы, состоящий из React фронтенда и FastAPI бэкенда.

## Структура проекта

- `backend/` — Бэкенд на FastAPI.
    - `app.py` — Основной сервер, шина событий (WebSocket).
    - `Dockerfile-dev` — Образ для разработки.
- `frontend/` — Фронтенд на React + Zustand.
    - `react/` — Исходный код React приложения.
    - `Dockerfile-dev` — Сборка фронтенда через Nginx.
- `tests/` — Интеграционные тесты всей системы.

## Особенности разработки (DEV)

### Пересборка фронтенда
В текущей конфигурации `web_ui_service/docker-compose-dev.yml` для сервиса `frontend` **отсутствует монтирование папки `src`** (volumes). 
Это означает, что любые изменения в React-коде (папка `frontend/react/src`) **не подхватываются автоматически**.

**Для применения изменений во фронтенде необходимо выполнить пересборку:**
```bash
# Пересобрать и перезапустить только фронтенд
(cd web_ui_service && docker compose -f docker-compose-dev.yml up -d --build frontend)
```

Либо полный перезапуск:
```bash
./stop-dev.sh
./start-dev.sh
```

### WebSocket взаимодействие
Фронтенд взаимодействует с бэкендом через WebSocket (`/ws/{session_id}`) для получения мгновенных обновлений:
- Системные статусы (progress events).
- Ход мыслей модели (reasoning/thought).
- Финальные ответы.

### Управление сессиями (v2.0)
- **Генерация ID**: Сессии создаются в формате `{username}_{random_hash}`.
- **Привязка**: Каждая сессия при создании регистрируется в `user_service` для привязки к владельцу.
- **Безопасность**: Пользователи видят только свои сессии. Роль `developer` дает доступ к системным и тестовым (`test_`) сессиям.

### Интеграционные тесты
Интеграционные тесты находятся в папке `web_ui_service/tests` и монтируются в контейнер бэкенда. Запуск:
```bash
docker exec web_ui_service-backend-dev uv run python /app/tests_integration/test_netrunner_scenarios.py --cfg /app/tests_integration/config-dev.json