# TODO: Фильтры безопасности, стоимости отдыха и сезонов на карте (frontend)

## Контекст

Проект: **`visa-map2-frontend`** (Next.js App Router, MapLibre), связка с **`visa-map2`**.

После бэкенда по безопасности и стоимости ([`todo_self_coast.md`](todo_self_coast.md)) и сезонам ([`todo_seasons.md`](todo_seasons.md)) на фронте нужны:

- фильтры в боковой панели;
- раскраска карты по выбранному режиму;
- попап страны с полями safety/cost при наличии данных.

Полный план реализации: см. `.cursor/plans/фильтры_и_раскраска_карты_8338cfe2.plan.md` (не редактировать).

## Задача

1. Расширить контракт **`GET /countries/geodata`**: в `properties` каждой фичи — `safety_level`, `cost_level`, `cost_per_day_usd` (для `match` в MapLibre без второго запроса).
2. Поднять состояние режима карты в **`page.tsx`**: `mapColorMode`, `seasonMonth`, множества активных уровней безопасности и стоимости.
3. Обновить **`FilterSidebar`**: секции «Безопасность», «Стоимость отдыха», «Сезонность» (месяц 1–12); снять заглушку цвета с season/budget; заглушки оставить для «Прямой рейс» / «Дальность перелёта».
4. Обновить **`VisaMap`**: режимы citizenship / safety / budget / season; слой сезонов из **`GET /country-seasons/{month}/geodata`**; обработка 404 без падения UI.
5. Обновить **`CountryPopup`** и при необходимости типы; **`README.md`** фронта.

## Шаги и критерии готовности

### Backend (`visa-map2`)

- [x] В `get_countries_geodata` и `get_country_geodata` добавить поля в `select` и в `properties` GeoJSON.
- [x] Сброс устаревшего кеша геоданных (новый ключ Redis `countries:geodata:v2` или `cache_delete` при обновлении стран).
- [ ] При появлении API/админки массового обновления стран — вызывать инвалидацию `GEODATA_KEY` из сервиса обновления.

### Frontend (`visa-map2-frontend`)

- [x] Файл типов режима: `app/types/map.ts` (`MapColorMode`).
- [x] `app/page.tsx`: состояние и пропсы в `FilterSidebar` и `VisaMap`.
- [x] `FilterSidebar.tsx`: группы фильтров, тогглы, выбор месяца, переключатель «цвет».
- [x] `VisaMap.tsx`: палитры, `applyAttributeColors`, сезонный source/layer, синхронизация с `coloringEnabled`.
- [x] `CountryPopup.tsx`: отображение `safety_level`, `cost_level`, `cost_per_day_usd`.
- [x] `README.md`: env, таблица режимов и эндпоинты.

### Проверки

- [x] `npm run lint` / `npm run build` (фронт).
- [ ] Ручная проверка в браузере: все режимы, смена месяца, паспорт, 404 сезонов.
- [ ] При росте размера GeoJSON сезонов — замер в DevTools; при необходимости планировать MVT/PMTiles (см. skill visa-map-frontend / patterns).

## Связанные файлы

| Область | Путь |
|--------|------|
| Геоданные стран | `visa-map2/app/services/country_service.py` |
| Кеш | `visa-map2/app/cache.py` (`GEODATA_KEY`) |
| Сезоны API | `visa-map2/app/routers/country_seasons.py` |
| Главная / состояние | `visa-map2-frontend/app/page.tsx` |
| Сайдбар | `visa-map2-frontend/app/components/FilterSidebar.tsx` |
| Карта | `visa-map2-frontend/app/components/VisaMap.tsx` |
| Попап | `visa-map2-frontend/app/components/CountryPopup.tsx` |

## Обновление memory-bank

После заметных доработок по этой фиче обновить при необходимости:

- `memory-bank/frontend.md` — режимы карты и API;
- `memory-bank/architecture.md` / `dataflow.md` — если меняется описание контрактов.
