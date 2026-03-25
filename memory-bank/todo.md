# Что сделано и что планируется

## ✅ Сделано
### Backend
- FastAPI + SQLAlchemy async + PostgreSQL/PostGIS + Redis
- 250 стран с геометрией, языками, доменами, переводами
- 39 402 визовых режима (Passport Index)
- Admin API с историей изменений
- confidence_level система (1/2/3)
- RSS мониторинг (23 источника, Google News агрегаторы)
- Инструмент обнаружения локальных источников
- Docker Compose для dev окружения

### Frontend
- Next.js 16 + MapLibre GL + Maptiler + Tailwind
- Интерактивная карта с окраской по визовым режимам
- Дропдаун выбора паспорта с поиском (250 стран)
- Карточка страны при клике
- Легенда
- Лонгрид структура страницы (в процессе)

## 🔄 В процессе

### Лонгрид UI — переработка главной страницы (план согласован)

**Цель:** главная страница как лонгрид. Первый экран — карта на 100vh, ниже — секции с подборками. Навбар фиксированный сверху.

**Компоненты для создания:**
- `app/components/Navbar.tsx` — фиксированный навбар с:
  - кнопкой профиля/меню (слева)
  - `CitizenshipFilter` (PassportSelect, всегда виден, нельзя убрать)
  - динамическими pill-фильтрами (предпочтения, бюджет, даты — UI-заглушки)
  - кнопкой "Фильтры" (справа) → открывает FiltersPanel
- `app/components/FiltersPanel.tsx` — drawer со всеми фильтрами (UI-заглушка)
- `app/components/DestinationsSection.tsx` — секция "Куда поехать"
- `app/components/DestinationCard.tsx` — карточка направления

**Изменения в существующих файлах:**
- `app/page.tsx` — поднять passport state, переписать в лонгрид (Navbar + секция карты + секции подборок)
- `app/globals.css` — убрать `overflow: hidden` с body
- `app/components/VisaMap.tsx` — убрать встроенный PassportSelect, принимать passport через props

**Порядок задач:**
1. Поднять passport state в page.tsx, передать в VisaMap через props
2. Создать Navbar с CitizenshipFilter
3. Создать FiltersPanel (drawer)
4. Переписать page.tsx в лонгрид, убрать overflow:hidden с body
5. Создать DestinationsSection + DestinationCard (данные из /countries)

## 📋 Планируется
### Backend
- Правило двух стран в RSS триггере
- Таблица очереди агрегатора (aggregator_queue)
- APScheduler для автозапуска мониторинга
- Парсинг МИД сайтов

### Frontend
- Детальная страница страны
- Статистика на главной (счётчики безвизовых и т.д.)
- Адаптив для мобильных
- SEO мета-теги

### Инфраструктура
- Деплой backend (VPS или Railway)
- Деплой frontend (Vercel)
- CI/CD через GitHub Actions