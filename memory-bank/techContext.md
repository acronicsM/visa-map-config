# Технический контекст

## Стек технологий

### Backend
| Компонент | Технология | Версия |
|-----------|-----------|--------|
| Язык | Python | 3.11 |
| Фреймворк | FastAPI | 0.111.0 |
| ORM | SQLAlchemy (async) | 2.0.30 |
| Геоданные | GeoAlchemy2 | 0.15.0 |
| Миграции | Alembic | 1.13.1 |
| БД | PostgreSQL + PostGIS | 16 + 3 |
| Кеш | Redis | — |
| Драйвер БД | asyncpg | 0.29.0 |
| HTTP клиент | httpx | — |
| RSS парсер | feedparser | — |
| HTML парсер | beautifulsoup4 | — |

### Frontend
| Компонент | Технология | Версия |
|-----------|-----------|--------|
| Фреймворк | Next.js (App Router) | 16.2.1 |
| Язык | TypeScript | — |
| Карта | MapLibre GL JS | — |
| Стили | Tailwind CSS | — |
| Картографический тайл | Maptiler (streets-v2) | — |

## Структура репозиториев

### Backend (`c:\ProjectVisaMap\visa-map2`)
```
visa-map2/
├── app/
│   ├── models/          # ORM: country, passport, visa_policy, rss_source, ...
│   ├── routers/         # Endpoints: admin.py, countries.py, visa_map.py
│   ├── schemas/         # Pydantic: admin.py, country.py, visa_policy.py, common.py
│   ├── services/        # Бизнес-логика
│   ├── main.py, config.py, database.py, cache.py, ...
├── alembic/             # Миграции БД
├── scripts/             # Скрипты загрузки данных
├── docker-compose.yml   # PostgreSQL + Redis
├── Dockerfile           # Production образ app
├── requirements.txt     # Python зависимости
└── geodata.json         # Кешированный GeoJSON (~5MB)
```

### Frontend (`c:\ProjectVisaMap\visa-map2-frontend`)
```
visa-map2-frontend/
├── app/
│   ├── components/      # VisaMap.tsx, PassportSelect.tsx, CountryPopup.tsx
│   ├── layout.tsx       # Root layout
│   ├── page.tsx         # Главная страница
│   └── globals.css      # Глобальные стили
├── public/              # Статические файлы
├── package.json
├── tsconfig.json
├── eslint.config.mjs    # ESLint + next/core-web-vitals
└── next.config.ts
```

## Инфраструктура разработки

### Порты
| Сервис | Порт |
|--------|------|
| Frontend (Next.js) | 3000 |
| Backend (FastAPI) | 8000 |
| PostgreSQL | 5442 (host) → 5432 (container) |
| Redis | 6379 |

### Docker Compose (только БД и Redis)
```bash
# Запуск инфраструктуры
docker compose up -d

# Подключение к БД
psql -h localhost -p 5442 -U visauser -d visamap
```

### Переменные окружения

**Backend (`.env`)**
```
DATABASE_URL=postgresql+asyncpg://visauser:visapass@localhost:5442/visamap
REDIS_URL=redis://localhost:6379
ADMIN_API_KEY=dev-secret-key
```

**Frontend (`.env.local`)**
```
NEXT_PUBLIC_MAPTILER_KEY=QSXDQ2qaHgSaUcWyPlkF
NEXT_PUBLIC_API_URL=http://localhost:8000
```

## Линтинг и форматирование

### Backend
- **Ruff** — линтинг и форматирование Python
- Длина строки: 88 символов (стиль Black)

### Frontend
- **ESLint** с `next/core-web-vitals`
- **Prettier** — форматирование TypeScript/TSX

## Ограничения и особенности

- Backend запускается напрямую (не в Docker), только БД и Redis в контейнерах
- Maptiler API ключ публичный (NEXT_PUBLIC_), допустимо для dev
- GeoJSON `geodata.json` закомиченный файл — кеш геоданных на диске
- `promoteId: 'iso2'` в MapLibre несовместим с `generateId` — использовать только один из них
