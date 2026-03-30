---
name: visa-map-frontend
description: >-
  Guides development and debugging of the VisaMap frontend in folder
  visa-map2-frontend: Next.js App Router, TypeScript, Tailwind CSS, MapLibre,
  PassportSelect, VisaMap, CountryPopup, API integration with backend.
  Use whenever the user works on frontend UI, page layout, components, hooks,
  styles, map behavior, API rendering, or mentions фронтенд, frontend,
  Next.js, App Router, Tailwind, TypeScript, React, map, MapLibre,
  visa-map2-frontend, layout.tsx, page.tsx, globals.css.
---

# VisaMap — фронтенд (Next.js)

Используй этот skill для задач в **`visa-map2-frontend/`**.

## Контекст проекта

- Фреймворк: **Next.js App Router** + **TypeScript**.
- UI: **Tailwind CSS**, минимум глобальных стилей в `app/globals.css`.
- Карта: **MapLibre GL**.
- Основные компоненты: `VisaMap`, `PassportSelect`, `CountryPopup`,
  `FilterSidebar`.
- Источник данных: backend API (`NEXT_PUBLIC_API_URL`, обычно `:8000`).

## Быстрые ориентиры по структуре

| Что | Где |
|-----|-----|
| Корень приложения | `visa-map2-frontend/app/` |
| Лейаут | `app/layout.tsx` |
| Главная | `app/page.tsx` |
| Компоненты | `app/components/` |
| Глобальные стили | `app/globals.css` |
| Конфиг линтинга | `eslint.config.mjs` |

## Правила изменений

1. Используй **TypeScript** и корректные типы пропсов/состояния.
2. Для стилей сначала выбирай **Tailwind классы** в JSX/TSX.
3. `globals.css` меняй только когда utility-классов недостаточно.
4. Для страниц используй только **App Router** (`app/`), не `pages/`.
5. Не ломай API-контракт с backend: проверяй поля ответов и fallback-ветки.
6. Не оставляй отладочные `console.log` без причины.

## Паттерн для UI-правок из браузера

Если пользователь приносит CSS-изменения из preview:

1. Найди элемент по `selector` / `className` / названию React-компонента.
2. Если это Tailwind-классы — меняй `className` в TSX, а не добавляй новый CSS.
3. Точно сопоставляй значения:
   - `2px` -> `0.5` scale (`gap-0.5`, `p-0.5`) или arbitrary `mx-[2px]` при необходимости.
   - `10px` -> `2.5` scale (`gap-2.5`).
4. Сохраняй специфичность и не затрагивай соседние блоки.
5. После правки проверь lint и визуально целевой блок.

## MapLibre и данные карты

- Инициализацию карты и слои меняй аккуратно в `VisaMap.tsx`.
- Не удаляй `promoteId`/`feature-state` механику без причины.
- Изменения визуализации виз делай так, чтобы:
  - корректно работали фильтры категорий,
  - hover/попап не деградировали,
  - не ломались сценарии при пустом API-ответе.

## Запуск и проверка

Из `visa-map2-frontend`:

- dev: `npm run dev`
- lint: `npm run lint`
- build check: `npm run build`

После существенных изменений UI:

1. Проверь страницу в браузере (`http://localhost:3000`).
2. Сверь состояние карты при смене паспорта и категорий.
3. Проверь, что нет TS/ESLint ошибок в измененных файлах.

## Что не делать

- Не смешивай фронтенд и бэкенд изменения в одной правке без необходимости.
- Не хардкодь URL API в компонентах, используй `NEXT_PUBLIC_API_URL`.
- Не добавляй тяжелые зависимости без явного запроса пользователя.

## Полезные ссылки внутри проекта

- `memory-bank/frontend.md` — обзор frontend архитектуры.
- `memory-bank/techContext.md` — стек, порты, env.
- `memory-bank/systemPatterns.md` — связь карты, API и UI.
