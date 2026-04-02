# Паттерны: геобэкенд (VisaMap / FastAPI)

Примеры ориентировочные; подгоняй под фактические типы и слой БД проекта.

## Валидация тела запроса как GeoJSON (после `geojson-pydantic`)

```python
from geojson_pydantic import FeatureCollection

# В роутере: body: FeatureCollection — Pydantic проверит структуру
```

## Потоковая отдача большого FeatureCollection

Идея: не собирать весь JSON в `str`; писать chunks (или использовать ORM/сырой курсор с батчами).

```python
import json
from collections.abc import AsyncIterator

from fastapi.responses import StreamingResponse

async def iter_feature_collection(
    features_batches: AsyncIterator[list[dict]],
) -> AsyncIterator[bytes]:
    yield b'{"type":"FeatureCollection","features":['
    first = True
    async for batch in features_batches:
        for feat in batch:
            if not first:
                yield b","
            first = False
            yield json.dumps(feat, separators=(",", ":")).encode()
    yield b"]}"

# return StreamingResponse(iter_feature_collection(...), media_type="application/geo+json")
```

## ETag для стабильного GeoJSON

```python
import hashlib

def weak_etag_from_bytes(payload: bytes) -> str:
    digest = hashlib.sha256(payload).hexdigest()[:16]
    return f'W/"{digest}"'

# В ответе: headers={"ETag": etag, "Cache-Control": "public, max-age=..."}
# При If-None-Match совпадении — 304
```

## PostGIS: только параметры, геометрия через bind

Не вставлять WKT в строку запроса; использовать функции SQLAlchemy/GeoAlchemy2 и bound parameters.

## Тесты геометрии

```python
from shapely.geometry import shape, mapping
from shapely import is_valid, make_valid

def test_polygon_valid_after_import():
    geo = {"type": "Polygon", "coordinates": [[[0, 0], [1, 0], [1, 1], [0, 0]]]}
    g = shape(geo)
    assert is_valid(g) or is_valid(make_valid(g))
```

Проверяй граничные случаи: пустые кольца, дубликаты вершин, антиподальные полигоны при упрощении.

## Docker (напоминание)

- Dev: `uvicorn app.main:app --reload` на хосте + `docker compose` для БД/Redis.
- Prod: multi-stage, non-root, минимальный runtime-образ; порядок слоёв — реже меняющиеся зависимости первыми.
