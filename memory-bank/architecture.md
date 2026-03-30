# Архитектура

## База данных (PostgreSQL + PostGIS)

### Основные таблицы
- **countries** — 250 стран, геометрия MultiPolygon/4326, iso2(PK), iso3,
  name_ru, name_en, name_native, flag_emoji, region, subregion, capital,
  geom, center_point, bbox_min/max_lat/lng, is_active,
  primary_language (ISO 639-3), language_name, all_languages (jsonb),
  country_tld, name_translations (jsonb), confidence_level,
  safety_level(safe/unsafe/dangerous), safety_note, safety_source,
  safety_updated_at, cost_level(low/medium/high), cost_per_day_usd,
  cost_updated_at

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
| GET | /country-seasons/{month}/geodata | GeoJSON сезонности за месяц |
| GET | /country-seasons/{iso2} | Сезоны по стране (1..12) |
| GET | /visa-map/{passport_iso2} | Карта виз (кеш 1h) |
| GET | /visa-map/{passport_iso2}/{dest_iso2} | Детали пары |
| PATCH | /admin/visa-policies/{id} | Обновить политику |
| POST | /admin/news-triggers | Создать триггер |
| GET | /admin/news-triggers | Список триггеров |
| PATCH | /admin/news-triggers/{id}/status | Обновить статус |

## Аутентификация Admin API
Header: X-Api-Key: dev-secret-key