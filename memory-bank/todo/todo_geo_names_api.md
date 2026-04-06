# TODO: Natural Earth полигоны без ISO2, названия/языки, API имён

## Контекст

Бэкенд `visa-map2`: полное покрытие shapefile Natural Earth 10m, обогащение restcountries, `GET /countries/names`, фильтр по языку.

## Задачи

- [x] Alembic + модель `natural_earth_admin0` (ключ `ADM0_A3`, `iso2` nullable, geom, bbox)
- [x] `scripts/import_geodata.py`: upsert всех записей NE + прежний merge в `countries` при наличии ISO2
- [x] `scripts/load_all_countries.py`: полный `name_translations` (translations + nativeName), статистика мультиязычия
- [x] `GET /countries/names` + Redis-кеш; расширить `CountryShort` (`official_language_codes`, `primary_language`); query `has_language` на `GET /countries`

**Локально:** `cd visa-map2 && alembic upgrade head`, затем при необходимости `python scripts/import_geodata.py` и `python scripts/load_all_countries.py`.

## Связанные файлы

| Область | Путь |
|--------|------|
| Модель NE | `visa-map2/app/models/natural_earth_admin0.py` |
| Миграция | `visa-map2/alembic/versions/` |
| Импорт гео | `visa-map2/scripts/import_geodata.py` |
| Загрузка стран | `visa-map2/scripts/load_all_countries.py` |
| API | `visa-map2/app/routers/countries.py`, `app/services/country_service.py`, `app/schemas/country.py` |
| Кеш | `visa-map2/app/cache.py` |

## Примечание (фронт)

Полигоны без ISO2 лежат в `natural_earth_admin0`; общий `/countries/geodata` без изменений до отдельной задачи на второй слой карты.
