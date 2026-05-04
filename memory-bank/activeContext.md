# Активный контекст

## Текущий фокус (4 мая 2026)

- **Документация**: синхронизированы `visa-map2/README.md`, `visa-map2-frontend/README.md` и файлы Memory Bank с фактическим API (travel-costs, country-seasons, отсутствие cost в GeoJSON).

## Реализовано недавно (кратко)

- **Матрица стоимостей** (`travel_cost_matrix`): `PUT /admin/travel-costs` (multipart JSON), публичные `GET /travel-costs/{home_iso2}?budget_tier=…`, `GET /travel-costs/score-bands`; на фронте режим раскраски «бюджет» завязан на паспорт (home) и tiers cheap/normal/expensive.
- **Безопасность**: импорт `PUT /admin/countries/safety-final-scores`, Redis-снимок баллов, маппинг в `countries.safety_level`, публичный `GET /countries/safety-final-scores`.
- **Сезоны**: API `country-seasons`, импорт GeoJSON по месяцам; в полном наборе входных файлов исторически встречались только месяцы 10–12.
- **Фронт**: `TravelCollections` + заглушка `/trip/[iso2]`; `VisaMap` отдаёт список iso2 под фильтры через `onMatchingIso2sChange`.

Подробный чеклист возможностей и бэклог — в `progress.md`.

## Активные решения и соображения

- Backend в рабочем состоянии для локальной разработки при поднятых Postgres/Redis.
- Главная (лонгрид) и `/trip/[iso2]` не завершены.

## Следующие шаги

1. При чистой БД: `alembic upgrade head`, загрузить данные (скрипты из README бэкенда), при необходимости импорт `travel_country_model_tier_means.json` через admin.
2. Догрузить season GeoJSON для месяцев 1–9 при появлении файлов и повторить импорт.
3. Завершить структуру главной (баннеры, подвал), наполнить `/trip/[iso2]`, страница `/country/[iso2]`, статистика на главной.

## Контекст для следующей сессии

- Репозитории в одном workspace: `visa-map2` (backend), `visa-map2-frontend` (frontend).
- Инфраструктура: `docker compose up -d` в каталоге бэкенда.
- Backend: `python -m uvicorn app.main:app --reload --port 8000`
- Frontend: `npm run dev`
- Dev-ключ Admin API задаётся в `.env` бэкенда (`API_KEY`); не использовать в проде.
