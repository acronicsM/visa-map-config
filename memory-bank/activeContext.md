# Активный контекст

## Текущий фокус (30 марта 2026)

Расширение модели стран полями безопасности и стоимости отдыха
(`safety_*`, `cost_*`) и базовое заполнение данных для всех стран.

## Последние изменения

- Инициализирован Memory Bank: созданы все обязательные core-файлы
- Добавлена миграция Alembic для новых полей `countries`:
  `safety_level`, `safety_note`, `safety_source`, `safety_updated_at`,
  `cost_level`, `cost_per_day_usd`, `cost_updated_at`
- Обновлены ORM-модель `Country` и схема `CountryDetail`
- Добавлен скрипт `scripts/seed_safety_cost.py` для рандомного заполнения
  safety/cost полей по всем странам
- Добавлен локальный Cursor skill
  `.cursor/skills/fastapi-router-py/SKILL.md` (адаптирован под структуру
  `visa-map2`: async FastAPI, `get_db`, `verify_api_key`, router->service)

## Активные решения и соображения

### Статус проекта
- Backend полностью функционален, API работает
- Frontend в базовом рабочем состоянии (карта, выбор паспорта, попап)
- Главная страница (лонгрид) не завершена

## Следующие шаги

1. Завершить лонгрид структуру главной страницы (баннеры, подвал)
2. Реализовать детальную страницу страны (`/country/[iso2]`)
3. Добавить статистику на главную
4. Настроить APScheduler для автозапуска RSS мониторинга
5. Подготовить деплой (Vercel + VPS/Railway)

## Контекст для следующей сессии

- Оба репозитория находятся в `c:\ProjectVisaMap\`
- Docker Compose нужно запустить перед стартом backend: `docker compose up -d`
- Backend запускается: `python -m uvicorn app.main:app --reload --port 8000`
- Frontend запускается: `npm run dev`
- Admin API ключ: `dev-secret-key` (только для dev окружения)
