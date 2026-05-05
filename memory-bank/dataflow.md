# Потоки данных

## Импорт safety scores (существующий)

```
safety_merged.json (by_iso2 → safety_final_score)
  → PUT /admin/countries/safety-final-scores
    → admin_service.store_safety_final_scores()
      → UPDATE countries.safety_level (safe/unsafe/dangerous) по порогам
      → Redis SET countries:safety_final_scores:v1 (persistent)
      → cache_delete(countries:geodata:v3)
```

## Импорт travel cost matrix (новый)

```
travel_country_model_tier_means.json (~50 MB)
  → PUT /admin/travel-costs (multipart/form-data)
    → travel_cost_service.import_travel_costs_from_file()
      → Streaming parse: countries.AD.countries[]
      → UPSERT travel_cost_matrix (home_iso2, dest_iso2, score_*, daily_cost_*,
        home_currency, income_daily, income_daily_usd, usd_to_home_rate)
        батчами по 1000 записей (insert().on_conflict_do_update)
      → cache_delete(countries:geodata:v3)
      → cache_delete_pattern(travel_costs:*)
```

## Чтение travel cost matrix (новый)

```
Frontend: паспорт выбран (home_iso2) + budgetTier выбран
  → GET /travel-costs/{home_iso2}?budget_tier=cheap|normal|expensive
    → travel_cost_service.get_travel_cost_map()
      → SELECT dest_iso2, score_{tier} FROM travel_cost_matrix WHERE home_iso2 = ?
      → Redis cache SET travel_costs:{home_iso2}:{tier} (TTL 24h)
    → Response: { "home_iso2": "RU", "budget_tier": "cheap", "scores": { "AD": 0.6676, ... } }
      → Frontend: VisaMap.applyTravelCostColors()
        → Paint expression MapLibre: match iso2 → color by score bucket
```

## Чтение точного бюджета путешественника

```
Frontend: паспорт выбран + открыт «Уточнить бюджет»
  → GET /travel-costs/{home_iso2}/exact-budget-data
    → travel_cost_service.get_exact_budget_data()
      → SELECT daily_cost_*, income_daily_usd, home_currency, usd_to_home_rate
        FROM travel_cost_matrix WHERE home_iso2 = ?
      → Redis cache SET travel_costs:{home_iso2}:exact_budget (TTL 24h)
  → GET /travel-costs/currencies?home_iso2={home_iso2}
    → supported popular currencies + default currency (passport currency if supported)
  → GET /travel-costs/fx-rate?currency={selected_currency}
    → Redis cache GET/SET fx:usd_to:{currency} (TTL 24h)
    → Frontend:
      selectedCurrency = home_currency if supported else USD
      budgetAmount = income_daily_usd * usd_to_selected_currency_rate * 10
      tripDays = 10
      dailyBudgetSelectedCurrency = budgetAmount / tripDays
      dailyBudgetUsd = dailyBudgetSelectedCurrency / usd_to_selected_currency_rate
      compare with DailyCost_cheap/normal/expensive per dest_iso2
      → VisaMap colors/filter by affordability band
```

## GeoJSON (countries/geodata)

```
GET /countries/geodata
  → country_service.get_countries_geodata()
    → SELECT iso2, name_ru, name_en, flag_emoji, region, bbox_*, safety_level, geom
    → Redis cache GET/SET countries:geodata:v3 (TTL 24h)
    → FeatureCollection без cost_level / cost_per_day_usd (v3)
```

## Карта виз

```
GET /visa-map/{passport_iso2}
  → visa_service.get_visa_map()
    → SELECT visa_category, max_stay_days FROM visa_policies WHERE passport_iso2 = ?
    → Redis cache GET/SET visa_map:{iso2} (TTL 1h)
```

## Сезоны

```
GET /country-seasons/{month}/geodata
  → country_season_service.get_season_geodata(month)
    → SELECT iso2, season, geom FROM country_seasons WHERE month = ?
    → GeoJSON FeatureCollection (без кеша в Redis)
```