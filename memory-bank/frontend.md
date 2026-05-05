# Frontend архитектура

## Структура
app/
├── page.tsx              # Главная страница (лонгрид)
├── globals.css           # Глобальные стили
├── trip/[iso2]/page.tsx  # Заглушка подборки путешествия по iso2
└── components/
    ├── VisaMap.tsx         # Карта; опционально `onMatchingIso2sChange(iso2[])`
    ├── FilterSidebar.tsx   # Фильтры главной (секция citizenship в UI — «Визовые режимы»)
    ├── TravelCollections.tsx # Лента направлений по фильтрам → ссылки на /trip/[iso2]
    ├── PassportSelect.tsx  # Дропдаун паспорта (в сайдбаре)
    └── CountryPopup.tsx    # Карточка страны при hover

## Карта (VisaMap.tsx)
- MapLibre GL JS с подложкой Maptiler (`NEXT_PUBLIC_MAPTILER_STYLE`, иначе `basic-v2`)
- GeoJSON из `/countries/geodata` с `promoteId: 'iso2'`; в `properties` есть
  `safety_level` (после admin-импорта merged JSON) — для фильтров/раскраски
- При необходимости точного балла безопасности: публичный `GET /countries/safety-final-scores`
- Бюджет:
  - «Ощущения бюджета»: `GET /travel-costs/{home_iso2}?budget_tier=cheap|normal|expensive`
    и пороги/подписи `GET /travel-costs/score-bands`;
  - «Уточнить бюджет»: `GET /travel-costs/{home_iso2}/exact-budget-data`,
    `GET /travel-costs/currencies` и `GET /travel-costs/fx-rate?currency=...`;
    автозаполнение `income_daily_usd * курс USD→выбранная валюта * 10`,
    пользователь может сменить валюту бюджета, сравнение всегда идёт в USD
    с `daily_cost_cheap|normal|expensive`.
- Слои: countries-fill, countries-border, countries-hover
- Цвета виз через match expression по iso2
- Hover эффект через feature-state
- При клике: загрузка /countries/{iso2} + поиск в visaDataRef
- После пересчёта раскраски вызывается `onMatchingIso2sChange` со списком iso2,
  проходящих тот же составной фильтр (виза, safety/cost, язык, сезон)

## Цвета визовых категорий
free:        #22c55e (зелёный)
voa:         #3b82f6 (синий)
evisa:       #eab308 (жёлтый)
embassy:     #f97316 (оранжевый)
restricted:  #ef4444 (красный)
unavailable: #6b7280 (серый)
unknown:     #1e293b (тёмно-синий)

## Цвета стоимости (относительный score)

Пороговые интервалы, подписи и цвета приходят с **`GET /travel-costs/score-bands`**; при ошибке загрузки на клиенте используются значения как в **`DEFAULT_TRAVEL_COST_SCORE_BANDS`** (аналог ниже для справки).

score < 0.5:    #22c55e (дешевле, чем дома)
0.5 ≤ score ≤ 1.0: #eab308 (примерно как дома)
score > 1.0:    #ef4444 (дороже, чем дома)
unknown:        #1e293b (нет данных)
## Переменные окружения (`.env.local`)

См. `visa-map2-frontend/.env.example`: `NEXT_PUBLIC_API_URL`, `NEXT_PUBLIC_MAPTILER_KEY`,
`NEXT_PUBLIC_MAPTILER_STYLE`, опционально `NEXT_PUBLIC_DEBUG_TRAVEL_COST_SCORE_BANDS`.

## Режимы раскраски карты (MapColorMode)
| Режим | Источник данных | Механика |
|-------|----------------|---------|
| citizenship | `GET /visa-map/{iso2}` | Цвет по visa_category (в сайдбаре — «Визовые режимы») |
| safety | `GET /countries/geodata` (properties.safety_level) | Цвет по safety_level с фильтром |
| budget | `/travel-costs/score-bands` + `/travel-costs/{home_iso2}?budget_tier=...` или `/travel-costs/{home_iso2}/exact-budget-data` | Цвет по budget bands; рельса cheap/normal/expensive — «Ощущения бюджета», раскрытый калькулятор — «Уточнить бюджет» |
| season | `GET /country-seasons/{month}/geodata` | Цвет по season_color + фильтр |
| language | `GET /countries` (official_language_codes) | Бинарная заливка |
| vacation | — | Заглушка |
| flight | — | Заглушка |