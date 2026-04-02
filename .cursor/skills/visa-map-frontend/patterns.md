# Паттерны: карта, GeoJSON, Docker (frontend)

## MapLibre + GeoJSON

- Источник: `map.addSource` / обновление `setData` — избегать полной замены огромного объекта при каждом чихе; по возможности **patch** данных или переключение **filter** / **feature-state**.
- **Кластеризация**: встроенные возможности MapLibre для `geojson` source (`cluster`, `clusterMaxZoom`, `clusterRadius`) — предпочтительный первый шаг.
- **Упрощение геометрии**: на клиенте — библиотеки вроде `@turf/simplify` (Douglas–Peucker); на очень больших наборах лучше **упрощать на сервере** или отдавать **тайлы**.
- **PMTiles / MVT**: при десятках/сотнях тысяч полигонов — рассмотреть тайловый пайплайн; фронт тогда запрашивает тайлы по zoom/extent, а не держит весь FC в памяти.

## React и перформанс

- Разделяй: контейнер карты (ref, lifecycle) vs презентация панелей.
- Тяжёлые производные от фич (группировка, индексы) — `useMemo` с явными зависимостями; не мемоизировать всё подряд.
- События карты (`moveend`, `zoom`) — **debounce/throttle** для запросов к API, если появятся динамические подгрузки по viewport.

## Типизация GeoJSON

- Импорт типов из `geojson`: `FeatureCollection`, `Feature`, `Geometry` — сужай под домен (например `Feature<Polygon>`).
- Ответы API — отдельные типы интерфейса (поля бэкенда ≠ сырой GeoJSON всегда).

## Docker (когда появится во фронтенде)

- **Multistage**: `deps` → `builder` → `runner` (или static export — отдельная схема).
- Кэш: копируй `package.json` + lock **до** `COPY .` для слоя npm ci/install.
- `NEXT_PUBLIC_*` нужны на **этапе build**, если зашиты в клиентский бандл.

## Context7

- Смотри актуальные доки для **Next.js App Router**, **maplibre-gl**, **@turf/***, при появлении — **react-map-gl** или аналоги.
