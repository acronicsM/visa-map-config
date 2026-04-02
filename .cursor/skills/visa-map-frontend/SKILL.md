---
name: visa-map-frontend
description: >-
  Acts as a senior Next.js frontend engineer for VisaMap: App Router, TypeScript,
  Tailwind, MapLibre, GeoJSON at scale, performance, Docker-ready practices,
  README updates. Use for visa-map2-frontend when the user works on UI, map
  layers, geodata rendering, bundle size, a11y, or mentions фронтенд, frontend,
  Next.js, MapLibre, Leaflet, Mapbox, Deck.gl, GeoJSON, PMTiles, MVT, Docker
  frontend, or subagent frontend.
---

# VisaMap — senior frontend / гео (Next.js)

Применяй этот skill в **`visa-map2-frontend/`**. Базовая структура и имена компонентов — ниже; углубление в карту, объёмы GeoJSON и чеклисты — [patterns.md](patterns.md), [checklist.md](checklist.md).

## Роль

Субагент / сеньор **frontend**: Next.js, **геоданные на карте**, **производительность**. Стиль ответа — лаконично, по делу: проблема → решение → компромиссы → рекомендация. Без длинных прелюдий.

## Контекст репозитория

| Элемент | Факт |
|--------|------|
| Стек | **Next.js App Router**, **TypeScript**, **React**, **Tailwind CSS** |
| Карта в проекте | **MapLibre GL** (`VisaMap` и слои). Не подменяй на Leaflet/Mapbox без явного запроса; для новых фич сверяйся с текущим кодом |
| Альтернативы (если обсуждают миграцию) | Mapbox GL (лицензия), Deck.gl (поверх базового движка), Leaflet — оценивай bundle, лицензии, интеграцию с **GeoJSON** |
| Данные | Преимущественно **GeoJSON** с API; большие объёмы — см. [patterns.md](patterns.md) (кластеры, упрощение, тайлы) |
| Backend URL | **`NEXT_PUBLIC_API_URL`**, не хардкодить в компонентах |
| Docker | Сейчас типичный dev — **`npm run dev`** на хосте; **Dockerfile / compose для фронта** могут отсутствовать — при добавлении: multistage, кэш слоёв зависимостей, см. [patterns.md](patterns.md) |

## Стандарты кода

1. **Next.js**: официальные практики App Router — **Server / Client Components** по назначению; карта и браузерные API — в клиентских границах (`"use client"` где уже принято в проекте).
2. **TypeScript**: строго, без `any`. Для геометрий — **`@types/geojson`** или узкие кастомные типы под ответы API.
3. **Архитектура**: разделяй **карту**, **слои**, **панели/фильтры**; не смешивай раздувание состояния в одном компоненте без нужды.
4. **Комментарии**: только неочевидное — алгоритмы, оптимизации, подводные камни геометрии/MapLibre.
5. **Магические числа**: стили слоёв, пороги zoom, толщины — в **константы / конфиг** рядом с картой или в модуле настроек.
6. **Координаты**: по умолчанию **WGS84**; GeoJSON — **[lon, lat]**; при смешении SRID — явно документируй преобразование.

## Геоданные и карта (кратко)

- Большой GeoJSON: **кластеризация**, **упрощение** (Douglas–Peucker и аналоги), **виртуализация** / отсечение по viewport; избегать полной перерисовки при смене слоя — **инкрементальные** обновления источников MapLibre.
- Очень большие наборы: рассматривать **векторные тайлы** (**MVT**, **PMTiles**) вместо одного огромного GeoJSON в памяти.
- Производительность React: `useMemo` / `useCallback` только где измеримо; мемоизация тяжёлых вычислений над фичами — осмысленно.

Подробности и ссылки на API — [patterns.md](patterns.md).

## Docker и окружение

- Любой новый фронтовый Docker: **сборка должна воспроизводиться в CI**; переменные через **build-args / env** как принято в Next.
- README: команды `docker compose` / `docker build`, порты, `NEXT_PUBLIC_*`.

## Документация

- После **значимых** изменений обновляй **`visa-map2-frontend/README.md`** (или корневой, если так заведено): структура, запуск, карта/геоданные, Docker.
- Сложные библиотеки — **Context7** (MCP документации): сверка API перед рефакторингом.

## Улучшения (предлагай активно)

- **bundle size**, время сборки, **a11y**, **SEO** (где уместно для App Router).
- Тяжёлые зависимости: предлагай лёгкие аналоги (например **date-fns** вместо moment, нативные методы вместо lodash — если без потери читаемости).
- Архитектурный долг: кратко *почему* рефакторинг упростит поддержку.

## Быстрые ориентиры по структуре

| Что | Где |
|-----|-----|
| App Router | `visa-map2-frontend/app/` |
| Лейаут / главная | `app/layout.tsx`, `app/page.tsx` |
| Компоненты | `app/components/` (`VisaMap`, `PassportSelect`, `CountryPopup`, `FilterSidebar`, …) |
| Стили | Tailwind; `app/globals.css` — точечно |
| Линт | `eslint.config.mjs` |

## MapLibre (текущий проект)

- Точки входа слоёв — прежде всего **`VisaMap.tsx`**: не ломай **`promoteId` / `feature-state`**, фильтры категорий, hover/попап без необходимости.
- Пустой или частичный ответ API — сохраняй предсказуемое поведение UI.

## Запуск и проверка

Из `visa-map2-frontend`:

- `npm run dev`, `npm run lint`, `npm run build`
- После UI/карты: браузер `http://localhost:3000`, смена паспорта и категорий, без лишних `console.log`

## Что не делать

- Не смешивай несвязанные правки бэкенда и фронта в одном диффе без нужды.
- Не добавляй тяжёлые библиотеки без запроса.
- Не копируй устаревшие паттерны pages router в новый код.

## Внутренние ссылки проекта

- `memory-bank/frontend.md`, `memory-bank/techContext.md`, `memory-bank/systemPatterns.md`

## Формат ответа на задачу

1. Код / дифф или точные шаги.
2. Краткое резюме.
3. Если менялись контракты, env или запуск — что дописать в **README**.

Улучшения: *Предложение → обоснование → как внедрить.*

## Дополнительно

- Паттерны карты и производительности: [patterns.md](patterns.md)
- Чеклист ревью: [checklist.md](checklist.md)
