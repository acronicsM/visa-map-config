---
name: visa-map-backend
description: >-
  Guides development and debugging of the VisaMap FastAPI backend in folder
  visa-map2: async SQLAlchemy 2, routers, services, Pydantic v2 schemas,
  Redis caching, Admin API with API key, Alembic migrations, Docker Compose
  for Postgres/Redis. Use whenever the user works on the backend, adds or
  changes API endpoints, models, services, schemas, migrations, cache, RSS,
  or mentions бэкенд, backend, FastAPI, visa-map2, SQLAlchemy, Alembic,
  PostgreSQL, PostGIS, Redis, admin API, uvicorn, pytest, or Ruff.
---

# VisaMap — бэкенд (FastAPI)

Используй этот skill при любых задачах в каталоге **`visa-map2/`** (не путать с `visa-map2-frontend/`).

## Расположение и стек

| Что | Где / что |
|-----|-----------|
| Корень API | `visa-map2/app/` |
| Точка входа | `app/main.py` — FastAPI, CORS, роутеры, OpenAPI + `x-api-key` для admin |
| Роутеры | `app/routers/` — `countries`, `visa_map`, `admin` |
| Сервисы | `app/services/` — бизнес-логика и запросы к БД |
| Модели ORM | `app/models/` |
| Схемы Pydantic | `app/schemas/` |
| Сессия БД | `app/database.py`, dependency `get_db` |
| Кеш Redis | `app/cache.py` |
| Конфиг | `app/config.py` (pydantic-settings) |
| Зависимости (ключ admin) | `app/dependencies.py` |
| Исключения | `app/exceptions.py` + регистрация в main |
| Миграции | `visa-map2/alembic/`, команды из корня `visa-map2` |
| Инфра dev | `visa-map2/docker-compose.yml` — Postgres + Redis |

Стек: **Python 3.11**, **FastAPI**, **SQLAlchemy 2 async**, **asyncpg**, **Pydantic v2**, **Alembic**, **PostgreSQL + PostGIS**, **Redis**.

## Архитектурный паттерн

1. **Роутер** (`routers/*.py`) — только HTTP: параметры пути/запроса, `Depends`, вызов сервиса, возврат схемы ответа.
2. **Сервис** (`services/*.py`) — запросы к БД, транзакции, вызовы кеша, бизнес-правила.
3. **Модель** (`models/*.py`) — таблицы SQLAlchemy.
4. **Схемы** (`schemas/*.py`) — вход/выход API, отдельно от ORM.

Новый эндпоинт: по возможности добавляй метод в существующий сервис или новый сервис, а не раздувая роутер.

## Правила кода (проект)

- Все эндпоинты — **`async def`**.
- Схемы — **Pydantic v2** (`model_config`, `Field`, и т.д.).
- Ошибки — **`HTTPException`** или кастомные исключения из `app/exceptions.py`, согласованные с хендлерами.
- Работа с БД — **асинхронная сессия** SQLAlchemy через `get_db`.
- Стиль: **PEP 8**, **Ruff**, длина строки **до 88 символов** (как в правилах репозитория).
- Не оставляй отладочные `print` без нужды.

## Кеширование

- GeoJSON и visa-map кешируются в Redis (см. `cache.py` и сервисы).
- При изменении данных через Admin API — **инвалидируй** соответствующие ключи (как уже сделано для visa-map по паспорту).

## Admin API

- Заголовок: **`X-Api-Key`** (в OpenAPI — `x-api-key`), значение из `.env` (`ADMIN_API_KEY` в dev часто `dev-secret-key`).
- Роуты с тегом admin защищены dependency с проверкой ключа.

## Локальный запуск (кратко)

1. Поднять Postgres + Redis: из `visa-map2` — `docker compose up -d` (порты см. `docker-compose.yml`; в memory-bank обычно Postgres **5442** на хосте).
2. Переменные в **`.env`** в `visa-map2` (как минимум `DATABASE_URL`, `REDIS_URL`, `ADMIN_API_KEY`).
3. Миграции: `alembic upgrade head` из каталога `visa-map2`.
4. Приложение: `python -m uvicorn app.main:app --reload --port 8000` из `visa-map2` (или с `cd visa-map2` и тем же модулем).

## Тесты и качество

- Критичную логику покрывай **pytest**-тестами в `visa-map2` (если тестов ещё нет в модуле — добавляй рядом с принятым в проекте расположением).
- После изменений — **Ruff** на затронутых файлах.

## Git (правило workspace)

Если в правилах указано использовать **GitKraken MCP** вместо shell для git — для коммитов/веток в этом проекте следуй этому правилу.

## Что не делать

- Не смешивай фронтенд (`visa-map2-frontend`) и бэкенд в одном изменении без необходимости; контракт API меняй осознанно и синхронизируй с фронтом отдельно.
- Не хардкодь секреты; только env / config.

## Быстрая навигация по файлам

- Список стран и geodata: `routers/countries.py` + `services/country_service.py`
- Карта виз: `routers/visa_map.py` + `services/visa_service.py`
- Админка и политики: `routers/admin.py` + `services/admin_service.py`

Подробная схема БД и эндпоинты — в `memory-bank/architecture.md` и `memory-bank/techContext.md` в корне workspace.
