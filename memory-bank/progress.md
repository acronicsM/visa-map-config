# Прогресс

## Что работает (реализовано)

### Backend
- [x] FastAPI приложение с async SQLAlchemy + PostgreSQL/PostGIS
- [x] Redis кеширование (GeoJSON 24h, visa-map 1h)
- [x] 250 стран с геометрией, языками, переводами, TLD
- [x] 39 402 визовых режима (из Passport Index CSV, confidence_level=3)
- [x] Admin API с аутентификацией по X-Api-Key
- [x] История изменений визовых политик (`visa_policy_history`)
- [x] Confidence level система (3 уровня достоверности)
- [x] RSS мониторинг (23 активных источника)
- [x] Обнаружение локальных источников (Google News RSS + TLD фильтр)
- [x] Docker Compose для dev окружения (PostgreSQL + Redis)
- [x] Alembic миграции
- [x] Скрипты загрузки данных (restcountries, Natural Earth, Passport Index)

### Frontend
- [x] Next.js 16 + MapLibre GL + Maptiler + Tailwind CSS
- [x] Интерактивная карта с окраской по визовым режимам
- [x] Дропдаун выбора паспорта с поиском (250 стран)
- [x] Карточка страны при клике (CountryPopup)
- [x] Легенда визовых категорий
- [x] Hover эффект на карте

## Что в процессе

- [ ] Лонгрид структура главной страницы (баннеры, подвал) — начато

## Что планируется

### Backend
- [ ] Правило двух стран в RSS триггере (триггер только если упомянуты обе страны)
- [ ] Таблица очереди агрегатора (`aggregator_queue`)
- [ ] APScheduler для автозапуска RSS мониторинга
- [ ] Парсинг МИД сайтов для confidence_level=1

### Frontend
- [ ] Детальная страница страны (`/country/[iso2]`)
- [ ] Статистика на главной (счётчики: безвизовых, е-визы и т.д.)
- [ ] Адаптивная вёрстка для мобильных устройств
- [ ] SEO мета-теги и Open Graph

### Инфраструктура
- [ ] Деплой backend (VPS или Railway)
- [ ] Деплой frontend (Vercel)
- [ ] CI/CD через GitHub Actions

## Известные проблемы и ограничения

- Данные Passport Index могут отставать на 1-3 месяца (confidence_level=3)
- 5 антарктических стран отключены (`is_active=false`)
- 11 стран без геометрии в Natural Earth shapefile
- RSS мониторинг запускается вручную скриптом (нет автозапуска)
- Maptiler ключ хардкодирован в `.env.local` (нужна ротация перед деплоем)
