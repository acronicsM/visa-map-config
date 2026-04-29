# Системные паттерны

## Общая архитектура

```
[Frontend Next.js :3000]
        |
        | HTTP (fetch)
        v
[Backend FastAPI :8000]
        |
   +----+----+
   |         |
   v         v
[PostgreSQL  [Redis]
 + PostGIS]   (cache)
```

## Backend — ключевые паттерны

### Структура приложения (FastAPI)
```
app/
├── main.py              # Точка входа, подключение роутеров, CORS
├── config.py            # Настройки через pydantic-settings
├── database.py          # Async SQLAlchemy engine + sessionmaker
├── dependencies.py      # FastAPI Depends (сессия БД, API-ключ)
├── cache.py             # Redis-клиент, декораторы кеширования
├── exceptions.py        # Кастомные исключения + обработчики
├── middleware.py        # Middleware (логирование запросов)
├── logging_config.py    # Настройка structlog/logging
├── models/              # SQLAlchemy ORM модели
├── schemas/             # Pydantic v2 схемы (request/response)
├── routers/             # FastAPI роутеры (endpoints)
└── services/            # Бизнес-логика (запросы к БД)
```

### Паттерн роутер → сервис → модель
- Роутер (`routers/`) принимает запрос, валидирует через Pydantic
- Сервис (`services/`) содержит бизнес-логику и SQL-запросы
- Модель (`models/`) описывает таблицы через SQLAlchemy ORM

### Кеширование
- GeoJSON (`/countries/geodata`): Redis, TTL=24h, ключ **`countries:geodata:v3`**
  (`GEODATA_KEY` в `cache.py`; в `properties` фич — `safety_level`)
- Справочник имён: **`countries:names:v1`**, TTL 24h
- Карта баллов безопасности (без TTL до перезаписи): **`countries:safety_final_scores:v1`**
  (`SAFETY_FINAL_SCORES_KEY`), запись через `cache_set_persistent`
- Visa map (`/visa-map/{iso2}`): Redis, TTL=1h, ключ `visa_map:{iso2}`
- При PATCH визовой политики — инвалидация `visa_map:{passport_iso2}`
- При **`PUT /admin/countries/safety-final-scores`** — инвалидация **`countries:geodata:v3`**
  после коммита в Postgres
- При **`PUT /admin/travel-costs`** — инвалидация **`countries:geodata:v3`** и
  **`travel_costs:*`** после UPSERT в Postgres

### Аутентификация Admin API
- Header: `X-Api-Key: <секрет из .env>`
- Dependency **`require_api_key`** в `dependencies.py` (настройка **`API_KEY`** в `config.py`)

### Confidence Level (доверие к данным)
```
3 → Автоматически (Passport Index CSV)
2 → Проверено модератором (Admin PATCH)
1 → Верифицировано по МИД/официальному источнику
```

## Frontend — ключевые паттерны

### Структура (Next.js App Router)
```
app/
├── layout.tsx           # Root layout
├── page.tsx             # Главная страница (лонгрид)
├── globals.css          # Глобальные стили (минимум)
└── components/
    ├── VisaMap.tsx       # Главный компонент карты (MapLibre GL)
    ├── PassportSelect.tsx # Дропдаун выбора паспорта с поиском
    └── CountryPopup.tsx  # Карточка страны при клике
```

### Паттерн карты (VisaMap.tsx)
1. Инициализация MapLibre GL с подложкой Maptiler streets-v2
2. Загрузка GeoJSON из `/countries/geodata` → source `countries` с `promoteId: 'iso2'`
3. Три слоя: `countries-fill`, `countries-border`, `countries-hover`
4. При выборе паспорта: запрос `/visa-map/{iso2}` → `setPaintProperty` с `match` expression
5. Hover: `setFeatureState` для слоя `countries-hover`
6. Клик: запрос `/countries/{iso2}` + поиск в `visaDataRef` → показ `CountryPopup`

### Именование компонента
`VisaMap` (не `Map`) — `Map` конфликтует со встроенным JS объектом в Next.js

## База данных — ключевые паттерны

### Геометрия
- MultiPolygon/SRID 4326 в PostGIS
- `unary_union` при импорте (AU, FR имеют несколько записей в shapefile)
- `ST_SimplifyPreserveTopology(geom, 0.05)` → GeoJSON ~5MB (вместо ~50MB)
- Для weather seasons используется отдельная таблица `country_seasons`
  с upsert по ключу (`iso2`, `month`) при импорте из GeoJSON

### История изменений
- `visa_policy_history` — автоматически при каждом PATCH через Admin API
- Хранит: старые значения, кто изменил, когда

### RSS мониторинг
- `rss_sources` → `source_type` определяет логику фильтрации
  - `news_agency`: фильтрация по `keywords` jsonb
  - `aggregator`: все новости релевантны (URL уже таргетирован)
  - `official`: официальные источники
- `news_triggers` → статусы: `new → reviewing → processed/ignored`

## Связи между компонентами

```
PassportSelect
    → onChange → VisaMap.loadVisaData(iso2)
        → GET /visa-map/{iso2}
        → setPaintProperty на слое countries-fill

VisaMap (клик на страну)
    → GET /countries/{iso2}
    → поиск в visaDataRef (текущие данные виз)
    → показ CountryPopup

Admin API (PATCH /admin/visa-policies/{id})
    → обновление visa_policies
    → запись в visa_policy_history
    → инвалидация Redis кеша visa_map:{iso2}

Admin API (PUT /admin/countries/safety-final-scores)
    → store_safety_final_scores в admin_service
    → ORM UPDATE countries по iso2 (safety_level из safety_final_score + пороги)
    → Redis: полная карта баллов (persistent key)
    → cache_delete(countries:geodata:v3)

Admin API (PUT /admin/travel-costs)
    → import_travel_costs_from_file в travel_cost_service
    → Streaming parse JSON (countries.home_iso2 → countries[])
    → UPSERT батчами 1000 записей в travel_cost_matrix
    → cache_delete(countries:geodata:v3) + cache_delete_pattern(travel_costs:*)

Country Seasons import (`scripts/import_country_seasons.py`)
    → чтение `seasons_month_1..12.geojson` из `INPUT_FOLDER_SEASONS`
    → нормализация геометрии Polygon -> MultiPolygon
    → upsert в `country_seasons` по (`iso2`, `month`)

Country Seasons API
    → GET /country-seasons/{month}/geodata (FeatureCollection)
    → GET /country-seasons/{iso2} (список сезонов по месяцам)
```
