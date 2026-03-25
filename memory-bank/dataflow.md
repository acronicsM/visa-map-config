# Потоки данных

## Загрузка данных (скрипты)
1. scripts/load_all_countries.py
   - Запрос к restcountries.com/v3.1/all (2 запроса из-за лимита полей)
   - Set Cover алгоритм для определения primary_language
   - Создаёт Country + Passport(regular) для каждой страны

2. scripts/import_geodata.py
   - Скачивает Natural Earth 10m shapefile (~30MB)
   - Объединяет геометрии через unary_union (важно для AU, FR и др.)
   - ST_SimplifyPreserveTopology для оптимизации

3. scripts/import_passport_index.py
   - CSV с ~39K визовых режимов для 199 паспортов
   - confidence_level=3 (автоматические данные)
   - Маппинг: visa free→free, voa→voa, eta/e-visa→evisa, visa required→embassy

4. scripts/seed_rss.py
   - Заполняет rss_sources и rss_keywords

## RSS мониторинг
scripts/rss_monitor.py:
- Загружает активные источники из БД
- news_agency: фильтрует по source.keywords (jsonb)
- aggregator: все новости релевантны (requires_filter=False)
- Определяет страны из текста по названиям из countries таблицы
- Создаёт NewsTrigger если новость релевантна
- Дедупликация по source_url

## Обнаружение источников
scripts/discover_sources.py:
- Google News RSS поиск по названию страны + ключевые слова
- Извлекает домены из тега <source url="...">
- Фильтр по ccTLD страны (country_tld)
- Показывает статус: уже в БД или НОВЫЙ
- Два режима: discovery (общие новости) и visa (визовая тематика)
- Результаты в source_discovery_log

## Confidence Level система
3 = Passport Index (автоматически)
2 = Проверено модератором (Admin API)
1 = Верифицировано по МИД/официальному источнику