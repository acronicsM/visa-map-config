# TODO: Импорт и API сезонов погоды по странам (`country_seasons`)

## Контекст
Проект: `visa-map2` (только backend, без изменений в `visa-map2-frontend`).

Есть env-переменная `INPUT_FOLDER_SEASONS` с путём к папке, где лежат
GeoJSON-файлы сезонов погоды по странам мира.

Формат имени файлов:
- `seasons_month_1.geojson`
- ...
- `seasons_month_12.geojson`

Итого 12 файлов, каждый файл соответствует одному месяцу.

## Задача
Реализовать хранение и чтение сезонов по странам:
- таблица `country_seasons` в БД (PostgreSQL + PostGIS),
- ORM-модель SQLAlchemy,
- миграция Alembic,
- схемы Pydantic,
- роутер для чтения данных,
- скрипт загрузки GeoJSON из `INPUT_FOLDER_SEASONS` в БД.

## Входной формат данных
Пример входного файла (`FeatureCollection`), где геометрия может быть как
`Polygon`, так и `MultiPolygon`:

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Polygon",
        "coordinates": [[[1.5208333333333333, 42.4375], [1.5208333333333333, 42.645833333333336], [1.7291666666666476, 42.5625], [1.5208333333333333, 42.4375]]]
      },
      "properties": {
        "iso2": "AD",
        "season": "cold",
        "month": 1
      }
    },
    {
      "type": "Feature",
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": [[[[55.145833333333336, 22.68750000000001], [52.60416666666665, 22.93750000000001], [51.68749999999999, 23.979166666666668], [51.60416666666665, 24.354166666666668], [51.92872155, 23.96474844], [52.341075066, 24.007513739], [52.60027103, 24.209540106], [53.581309441, 24.048529364], [54.122569207, 24.14472077], [56.145833333333336, 26.062500000000004], [55.145833333333336, 22.68750000000001]]]]
      },
      "properties": {
        "iso2": "XX",
        "season": "pleasant",
        "month": 2
      }
    }
  ]
}
```

## Шаг 1 - Миграция Alembic
Создать миграцию:

```bash
alembic revision -m "add country_seasons table"
```

В миграции создать таблицу `country_seasons` с полями:
- `id` (UUID PK),
- `country_id` (UUID FK -> `countries.id`, `ondelete=CASCADE`),
- `iso2` (String(2), индекс),
- `month` (Integer, диапазон 1..12),
- `season` (String(32), индекс),
- `geom` (Geometry(MULTIPOLYGON, 4326)),
- `created_at`, `updated_at` (timezone-aware).

Добавить ограничения:
- уникальность (`iso2`, `month`),
- check-constraint для `month` (`month BETWEEN 1 AND 12`).

Применить миграцию:

```bash
alembic upgrade head
```

## Шаг 2 - ORM модель
Добавить модель `CountrySeason` в `app/models/country_season.py`:
- `__tablename__ = "country_seasons"`,
- типы и поля соответствуют миграции,
- связь с `Country` (many-to-one),
- PostGIS-тип через `geoalchemy2.Geometry`.

Обновить `app/models/__init__.py` и связь в `Country`, если в проекте
используется явный реестр моделей/relationships.

## Шаг 3 - Схемы Pydantic
Добавить схемы в `app/schemas/country_season.py`:
- `CountrySeasonItem` (iso2, month, season),
- `CountrySeasonGeoFeature` (GeoJSON feature),
- `CountrySeasonGeoResponse` (FeatureCollection),
- `CountrySeasonByCountryResponse` (список сезонов по стране).

При необходимости подключить схемы в `app/schemas/__init__.py`.

## Шаг 4 - Роутер чтения данных
Создать `app/routers/country_seasons.py`:

```text
GET /country-seasons/{month}/geodata
GET /country-seasons/{iso2}
```

Требования:
- эндпоинт по месяцу возвращает GeoJSON `FeatureCollection`,
- эндпоинт по стране возвращает сезоны по 12 месяцам,
- валидация параметров (`month` 1..12, `iso2` в верхнем регистре),
- корректные `HTTPException` (404, если данных нет).

Подключить роутер в `app/main.py`.

## Шаг 5 - Скрипт загрузки данных
Создать скрипт `scripts/import_country_seasons.py`.

Скрипт должен:
1. Считать `INPUT_FOLDER_SEASONS` из env.
2. Пройти по `seasons_month_1..12.geojson`.
3. Для каждой feature прочитать `iso2`, `season`, `month`, `geometry`.
4. Нормализовать геометрию:
   - `Polygon` -> `MultiPolygon`,
   - `MultiPolygon` оставить как есть.
5. Найти страну по `iso2` в таблице `countries`.
6. Выполнить upsert по ключу (`iso2`, `month`).
7. Вывести итоговую статистику:
   - сколько файлов обработано,
   - сколько записей вставлено,
   - сколько обновлено,
   - сколько пропущено (неизвестный `iso2` / невалидная геометрия).

## Шаг 6 - Проверка
```bash
# Миграции
alembic upgrade head

# Импорт сезонов
python scripts/import_country_seasons.py

# Быстрая проверка количества
docker exec -it visamap_postgres psql -U visauser -d visamap -c "
SELECT month, count(*) FROM country_seasons GROUP BY month ORDER BY month;"
```

Проверить API:
- `GET /country-seasons/1/geodata`
- `GET /country-seasons/RU`

## Шаг 7 - Обновить memory-bank
После реализации обновить:
- `memory-bank/architecture.md` (новая таблица `country_seasons` и API),
- `memory-bank/systemPatterns.md` (паттерн загрузки и чтения сезонов),
- `memory-bank/progress.md` (статус выполнения).

## Критерии готовности
- [ ] Создана и применена миграция `country_seasons`
- [ ] Добавлена ORM-модель и связи
- [ ] Добавлены Pydantic-схемы
- [ ] Добавлен и подключен роутер
- [ ] Реализован скрипт импорта из `INPUT_FOLDER_SEASONS`
- [ ] Данные загружаются минимум для 12 месяцев
- [ ] Эндпоинты возвращают корректный ответ
- [ ] Обновлены файлы memory-bank
