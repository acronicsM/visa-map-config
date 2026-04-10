# Активный контекст

## Текущий фокус (10 апреля 2026)

Связка **безопасность**: импорт merged JSON → Redis + маппинг в `countries.safety_level`
для фильтров и GeoJSON; документация в `visa-map2/README.md`. Параллельно —
лонгрид главной и подборки путешествий.

## Последние изменения

- **Backend — импорт safety scores**: `PUT /admin/countries/safety-final-scores`
  (JSON `by_iso2`, поле `safety_final_score`), защита `X-Api-Key`; карта баллов
  в Redis (`countries:safety_final_scores:v1`, `cache_set_persistent`); обновление
  в Postgres `safety_level`, `safety_updated_at`, `safety_source` через ORM-`UPDATE`
  по `iso2` (отказ от `jsonb_to_recordset` в сыром SQL — с asyncpg не обновляло строки);
  пороги `SAFETY_SCORE_SAFE_MIN` / `SAFETY_SCORE_UNSAFE_MIN` в `app/config.py` + `.env`;
  `GET /countries/safety-final-scores` (публично); после импорта — `cache_delete(GEODATA_KEY)`
- Сервис маппинга: `app/services/safety_level_mapping.py`
- Обновлён **`visa-map2/README.md`** (эндпоинты, импорт, env, структура репозитория)
- Главная: `TravelCollections` показывает страны, проходящие составной фильтр
  карты; `VisaMap` отдаёт список через `onMatchingIso2sChange`; добавлена
  заглушка `/trip/[iso2]` (перелёты/отели/инфо — позже)
- Инициализирован Memory Bank: созданы все обязательные core-файлы
- Добавлена миграция Alembic для новых полей `countries`:
  `safety_level`, `safety_note`, `safety_source`, `safety_updated_at`,
  `cost_level`, `cost_per_day_usd`, `cost_updated_at`
- Обновлены ORM-модель `Country` и схема `CountryDetail`
- Добавлен скрипт `scripts/seed_safety_cost.py` для рандомного заполнения
  safety/cost полей по всем странам
- Реализован модуль `country_seasons` в backend:
  - миграция `a1b2c3d4e5f6_add_country_seasons_table.py`
  - ORM-модель `CountrySeason` + связь в `Country`
  - Pydantic-схемы `app/schemas/country_season.py`
  - API-роутер `GET /country-seasons/{month}/geodata`,
    `GET /country-seasons/{iso2}`
  - скрипт импорта `scripts/import_country_seasons.py`
- Применена миграция `alembic upgrade head`
- Выполнен импорт сезонов: в текущем наборе найдены файлы только
  для месяцев 10-12 (месяцы 1-9 отсутствуют в `INPUT_FOLDER_SEASONS`)
- Добавлен локальный Cursor skill
  `.cursor/skills/fastapi-router-py/SKILL.md` (адаптирован под структуру
  `visa-map2`: async FastAPI, `get_db`, `verify_api_key`, router->service)

## Активные решения и соображения

### Статус проекта
- Backend полностью функционален, API работает
- Frontend в базовом рабочем состоянии (карта, выбор паспорта, попап)
- Главная страница (лонгрид) не завершена

## Следующие шаги

1. Догрузить season GeoJSON для месяцев 1..9 и повторно выполнить импорт
2. Добавить кеширование для `GET /country-seasons/{month}/geodata` (опционально)
3. Фронт: явно использовать `safety_level` / при необходимости `safety-final-scores` в фильтрах и легенде
4. Завершить лонгрид структуру главной страницы (баннеры, подвал)
5. Наполнить `/trip/[iso2]` (авиа, отели, справка) и связать с API при необходимости
6. Реализовать детальную страницу страны (`/country/[iso2]`)
7. Добавить статистику на главную

## Контекст для следующей сессии

- Оба репозитория находятся в `c:\ProjectVisaMap\`
- Docker Compose нужно запустить перед стартом backend: `docker compose up -d`
- Backend запускается: `python -m uvicorn app.main:app --reload --port 8000`
- Frontend запускается: `npm run dev`
- Admin API ключ: `dev-secret-key` (только для dev окружения)
