# Frontend архитектура

## Структура
app/
├── page.tsx              # Главная страница (лонгрид)
├── globals.css           # Глобальные стили
└── components/
    ├── VisaMap.tsx        # Главный компонент карты
    ├── PassportSelect.tsx # Дропдаун выбора паспорта с поиском
    └── CountryPopup.tsx   # Карточка страны при клике

## Карта (VisaMap.tsx)
- MapLibre GL JS с подложкой Maptiler streets-v2 (светлая)
- GeoJSON из /countries/geodata с promoteId: 'iso2'
- Слои: countries-fill, countries-border, countries-hover
- Цвета виз через match expression по iso2
- Hover эффект через feature-state
- При клике: загрузка /countries/{iso2} + поиск в visaDataRef

## Цвета визовых категорий
free:        #22c55e (зелёный)
voa:         #3b82f6 (синий)
evisa:       #eab308 (жёлтый)
embassy:     #f97316 (оранжевый)
restricted:  #ef4444 (красный)
unavailable: #6b7280 (серый)
unknown:     #1e293b (тёмно-синий)

## ENV переменные (.env.local)
NEXT_PUBLIC_MAPTILER_KEY=QSXDQ2qaHgSaUcWyPlkF
NEXT_PUBLIC_API_URL=http://localhost:8000