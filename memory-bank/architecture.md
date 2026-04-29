# Архитектура

## База данных (PostgreSQL + PostGIS)

### Основные таблицы
- **countries** — 250 стран, геометрия MultiPolygon/4326, iso2(PK), iso3,
  name_ru, name_en, name_native, flag_emoji, region, subregion, capital,
  geom, center_point, bbox_min/max_lat/lng, is_active,
  primary_language (ISO 639-3), language_name, all_languages (jsonb),
  country_tld, name_translations (jsonb), confidence_level,
  safety_level(safe/unsafe/dangerous), safety_note, safety_source,
  safety_updated_at.
  При `PUT /admin/countries/safety-final-scores` числа из merged JSON пишутся в Redis
  (`countries:safety_final_scores:v1`) и маппятся в **safety_level** (safe/unsafe/dangerous)
  в Postgres по порогам `SAFETY_SCORE_SAFE_MIN` / `SAFETY_SCORE_UNSAFE_MIN` в `.env`.

- **passports** — id, country_id(FK), name_ru, type(regular/diplomatic/service)

- **visa_policies** — id, passport_id(FK), destination_id(FK),
  visa_category(free/voa/evisa/embassy/restricted/unavailable),
  max_stay_days, visa_validity_days, processing_days, fee_usd,
  conditions(jsonb), source_url, verified_by, verified_at,
  confidence_level(1=МИД/2=модератор/3=авто), confidence_note

- **visa_policy_history** — история изменений visa_policies

- **news_triggers** — триггеры из RSS мониторинга,
  status(new/reviewing/processed/ignored), affected_countries(jsonb)

- **rss_sources** — источники RSS, source_type(news_agency/aggregator/official),
  requires_filter, fetch_frequency, keywords(jsonb), country_id(FK)

- **rss_keywords** — глобальный словарь ключевых слов по языкам

- **source_discovery_log** — результаты обнаружения локальных источников

- **country_seasons** — сезонность погоды по странам:
  id(UUID PK), country_id(FK -> countries.id, CASCADE), iso2, month(1..12),
  season, geom(MultiPolygon/4326), created_at, updated_at,
  уникальность (iso2, month)

- **travel_cost_matrix** — матрица стоимости путешествия (home_iso2 x dest_iso2 x budget_tier):
  id(UUID PK), home_iso2(VARCHAR(2)), dest_iso2(VARCHAR(2)),
  score_cheap(NUMERIC(10,4)), score_normal(NUMERIC(10,4)),
  score_expensive(NUMERIC(10,4)), daily_cost_cheap(NUMERIC(10,4)),
  daily_cost_normal(NUMERIC(10,4)), daily_cost_expensive(NUMERIC(10,4)),
  created_at, updated_at,
  уникальность (home_iso2, dest_iso2)

## Данные
- 250 стран (245 активных, 5 антарктических отключены)
- 239 стран с геометрией (Natural Earth 10m shapefile)
- 39 402 визовых режима (Passport Index датасет, confidence_level=3)
- 23 активных RSS источника
- Геометрия: ST_SimplifyPreserveTopology(geom, 0.05) для GeoJSON

## API эндпоинты
| Метод | URL | Описание |
|-------|-----|----------|
| GET | /health | Статус сервера |
| GET | /countries | Список стран |
| GET | /countries/geodata | GeoJSON (кеш 24h) |
| GET | /countries/{iso2} | Детали страны |
| GET | /countries/safety-final-scores | Карта iso2 → safety_final_score (Redis) |
| GET | /country-seasons/{month}/geodata | GeoJSON сезонности за месяц |
| GET | /country-seasons/{iso2} | Сезоны по стране (1..12) |
| GET | /visa-map/{passport_iso2} | Карта виз (кеш 1h) |
| GET | /visa-map/{passport_iso2}/{dest_iso2} | Детали пары |
| GET | /travel-costs/{home_iso2} | Матрица стоимостей по budget_tier (кеш 24h) |
| PUT | /admin/travel-costs | Загрузка travel_country_model_tier_means.json (multipart) |
| PUT | /admin/countries/safety-final-scores | Merged JSON (`by_iso2`) → Redis + Postgres `safety_level` + сброс кеша geodata |
| PATCH | /admin/visa-policies/{id} | Обновить политику |
| POST | /admin/news-triggers | Создать триггер |
| GET | /admin/news-triggers | Список триггеров |
| PATCH | /admin/news-triggers/{id}/status | Обновить статус |

## Аутентификация Admin API
Header: X-Api-Key: dev-secret-key