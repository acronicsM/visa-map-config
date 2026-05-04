# Технический стек

## Backend
- Python 3.11
- FastAPI 0.111.0
- SQLAlchemy 2.0.30 (async)
- GeoAlchemy2 0.15.0
- Alembic 1.13.1
- PostgreSQL 16 + PostGIS 3
- Redis (кеш GeoJSON 24h, visa-map 1h; persistent snapshot `safety_final_scores`)
- asyncpg 0.29.0
- httpx, feedparser, beautifulsoup4
- Docker (только для БД и Redis, не для app)
- Порт PostgreSQL: 5442:5432

## Frontend
- Next.js 16.2.1 (App Router, TypeScript)
- MapLibre GL JS
- Tailwind CSS
- Maptiler (`NEXT_PUBLIC_MAPTILER_STYLE`, по умолчанию в приложении — `basic-v2`)

## Инфраструктура
- Docker Compose (postgres + redis)
- БД: visauser/visapass/visamap
- Redis: localhost:6379
- Backend API: http://localhost:8000
- Frontend: http://localhost:3000