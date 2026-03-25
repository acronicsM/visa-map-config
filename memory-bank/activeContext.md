# Активный контекст

## Текущий фокус (25 марта 2026)

Инициализация Memory Bank — приведение документации в соответствие со стандартной структурой (productContext, systemPatterns, techContext, progress, activeContext).

## Последние изменения

- Инициализирован Memory Bank: созданы все обязательные core-файлы
- Существующие файлы (architecture.md, techstack.md, frontend.md, dataflow.md, decisions.md, todo.md) сохранены как дополнительный контекст

## Активные решения и соображения

### Локальные пути (важно!)
Локальные репозитории находятся в `c:\ProjectVisaMap\`, а не в `D:\Project_VisaMap\` как указано в `projectbrief.md`. Нужно обновить projectbrief.md.

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
- Backend запускается: `uvicorn app.main:app --reload --port 8000`
- Frontend запускается: `npm run dev`
- Admin API ключ: `dev-secret-key` (только для dev окружения)
