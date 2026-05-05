# VisaMap — workspace

В этом каталоге собраны **два связанных репозитория** одного продукта: бэкенд (FastAPI) и фронтенд (Next.js). Удобно открывать их одним workspace в Cursor.

## Репозитории

| Часть | Папка | GitHub |
|--------|--------|--------|
| Backend | `visa-map2/` | [acronicsM/visa-map2](https://github.com/acronicsM/visa-map2) |
| Frontend | `visa-map2-frontend/` | [acronicsM/visa-map2-frontend](https://github.com/acronicsM/visa-map2-frontend) |

Подробные инструкции по установке и API — в README каждого подпроекта:

- [visa-map2/README.md](./visa-map2/README.md) — Postgres/PostGIS, Redis, Alembic, Admin API, импорт данных
- [visa-map2-frontend/README.md](./visa-map2-frontend/README.md) — переменные окружения, карта MapLibre, режимы раскраски

## Быстрый ориентир (локально)

1. В `visa-map2/`: `docker compose up -d`, виртуальное окружение Python 3.11, `alembic upgrade head`, скрипты загрузки из README бэкенда, `uvicorn` на порту **8000**.
2. В `visa-map2-frontend/`: `npm install`, `.env.local` из `.env.example`, `npm run dev` на порту **3000**.

## Документация для разработки

В `memory-bank/` лежит согласованное описание архитектуры, потоков данных и текущего статуса (удобно для продолжения работы в новых сессиях). Начните с `memory-bank/projectbrief.md`, `memory-bank/activeContext.md` и `memory-bank/progress.md`.
