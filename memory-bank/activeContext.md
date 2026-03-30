# Активный контекст

## Текущий фокус (30 марта 2026)

Реализация backend-модуля сезонности погоды по странам (`country_seasons`):
таблица, API чтения и импорт GeoJSON по месяцам.

## Последние изменения

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
3. Завершить лонгрид структуру главной страницы (баннеры, подвал)
4. Реализовать детальную страницу страны (`/country/[iso2]`)
5. Добавить статистику на главную

## Контекст для следующей сессии

- Оба репозитория находятся в `c:\ProjectVisaMap\`
- Docker Compose нужно запустить перед стартом backend: `docker compose up -d`
- Backend запускается: `python -m uvicorn app.main:app --reload --port 8000`
- Frontend запускается: `npm run dev`
- Admin API ключ: `dev-secret-key` (только для dev окружения)
