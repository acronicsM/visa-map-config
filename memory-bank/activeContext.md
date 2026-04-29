# Активный контекст

## Текущий фокус (29 апреля 2026)

Реализация **матрицы стоимостей путешествия**: замена скалярных `cost_level` / `cost_per_day_usd`
в `countries` на таблицу `travel_cost_matrix` (home_iso2 x dest_iso2 x budget_tier).
- Admin endpoint `PUT /admin/travel-costs` для загрузки JSON через multipart.
- Public endpoint `GET /travel-costs/{home_iso2}?budget_tier=...` для получения score map.
- Frontend: фильтр стоимости теперь зависит от выбранного паспорта (home_iso2);
  раскраска карты по score buckets (< 0.5 зелёный, 0.5–1.0 жёлтый, > 1.0 красный).

## Последние изменения

- **Backend — матрица стоимостей**:
  - Удалены `cost_level`, `cost_per_day_usd`, `cost_updated_at` из `countries` (Alembic миграция).
  - Создана таблица `travel_cost_matrix` с полями `score_cheap`, `score_normal`, `score_expensive`,
    `daily_cost_*` и индексами `(home_iso2)`, `(home_iso2, dest_iso2)`.
  - `PUT /admin/travel-costs` — загрузка `travel_country_model_tier_means.json` (~50 MB)
    через `UploadFile` (multipart), валидация content-type и размера, парсинг JSON,
    UPSERT батчами по 1000 записей (`insert().on_conflict_do_update`),
    инвалидация Redis (`GEODATA_KEY`, `travel_costs:*`).
  - `GET /travel-costs/{home_iso2}?budget_tier=cheap|normal|expensive` — SELECT из
    `travel_cost_matrix` WHERE `home_iso2 = ?`, кеш Redis (`travel_costs:{home_iso2}:{tier}`, TTL 24h).
  - GeoJSON properties v3: убраны `cost_level` / `cost_per_day_usd`.

- **Frontend — фильтр бюджета**:
  - `FilterSidebar`: заменён набор чекбоксов уровней `cost_level` на выбор одного
    `budgetTier` (cheap / normal / expensive). При выборе автоматически включается
    режим раскраски «budget».
  - `VisaMap`: при смене паспорта + `budgetTier` вызывается `GET /travel-costs/{home_iso2}`;
    scores мержатся в paint expression MapLibre (match по iso2).
  - `CountryPopup`: убраны `cost_level` / `cost_per_day_usd`; при наличии `cost_score`
    показывается текст («Дешевле, чем дома» / «Примерно как дома» / «Дороже, чем дома»).
  - `page.tsx`: состояние `budgetTier` (string | null); `useEffect` загружает
    travel-costs данные при изменении `passport` или `budgetTier`.

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

1. Запустить `alembic upgrade head` (миграция drop cost + create travel_cost_matrix)
2. Загрузить `travel_country_model_tier_means.json` через `PUT /admin/travel-costs`
3. Проверить `GET /travel-costs/RU?budget_tier=cheap` в браузере
4. Догрузить season GeoJSON для месяцев 1..9 и повторно выполнить импорт
5. Завершить лонгрид структуру главной страницы (баннеры, подвал)
6. Наполнить `/trip/[iso2]` (авиа, отели, справка) и связать с API при необходимости
7. Реализовать детальную страницу страны (`/country/[iso2]`)
8. Добавить статистику на главную

## Контекст для следующей сессии

- Оба репозитория находятся в `c:\ProjectVisaMap\`
- Docker Compose нужно запустить перед стартом backend: `docker compose up -d`
- Backend запускается: `python -m uvicorn app.main:app --reload --port 8000`
- Frontend запускается: `npm run dev`
- Admin API ключ: `dev-secret-key` (только для dev окружения)
