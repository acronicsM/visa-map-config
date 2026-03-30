# TODO: Добавить safety_level и cost_level в таблицу countries

## Контекст
Проект visa-map2. Backend: FastAPI + SQLAlchemy async + PostgreSQL.
Читай memory-bank/ для полного контекста проекта.

## Задача
Добавить два новых поля в таблицу `countries`:
- `safety_level` — уровень безопасности страны (safe/unsafe/dangerous)
- `safety_note` - Комментарий к уровню безопастности, если есть
- `safety_source` - Ссылка на истоник установления уровня безопастности, если есть
- `safety_updated_at` - дата обновления уровня безопастости
- `cost_level` — стоиомсть отдыха (low/medium/high)
- `cost_per_day_usd` - Стоимость 1 дня отдыха в USD 
- `cost_updated_at` - дата обновления стоимости отдыха

## Шаг 1 — Миграция Alembic

Создай новую миграцию:
```bash
alembic revision -m "add safety_level and cost_level to countries"
```

В файле миграции реализуй:
```python
def upgrade() -> None:
    op.add_column('countries', sa.Column(
        'safety_level',
        sa.String(20),
        nullable=True,
        comment='safe / unsafe / dangerous'
    ))
    op.add_column('countries', sa.Column(
        'safety_note', sa.Text(), nullable=True
    ))
    op.add_column('countries', sa.Column(
        'safety_source', sa.Text(), nullable=True
    ))
    op.add_column('countries', sa.Column(
        'safety_updated_at', sa.DateTime(timezone=True), nullable=True
    ))
    op.add_column('countries', sa.Column(
        'cost_level',
        sa.String(20),
        nullable=True,
        comment='low / medium / high'
    ))
    op.add_column('countries', sa.Column(
        'cost_per_day_usd', sa.Integer(), nullable=True
    ))
     op.add_column('countries', sa.Column(
        'cost_updated_at', sa.DateTime(timezone=True), nullable=True
    ))

def downgrade() -> None:
    op.drop_column('countries', 'cost_updated_at')
    op.drop_column('countries', 'cost_per_day_usd')
    op.drop_column('countries', 'cost_level')
    op.drop_column('countries', 'safety_updated_at')
    op.drop_column('countries', 'safety_source')
    op.drop_column('countries', 'safety_note')
    op.drop_column('countries', 'safety_level')
```

Применить миграцию:
```bash
alembic upgrade head
```

## Шаг 2 — Обновить модель Country

Файл: `app/models/country.py`

Добавить поля в класс Country после существующих полей:
```python
safety_level: Mapped[str | None] = mapped_column(String(20), nullable=True)
safety_note: Mapped[str | None] = mapped_column(Text, nullable=True)
safety_source: Mapped[str | None] = mapped_column(Text, nullable=True)
safety_updated_at: Mapped[datetime | None] = mapped_column(DateTime(timezone=True), nullable=True)
cost_level: Mapped[str | None] = mapped_column(String(20), nullable=True)
cost_per_day_usd: Mapped[int | None] = mapped_column(Integer, nullable=True)
cost_updated_at: Mapped[datetime | None] = mapped_column(DateTime(timezone=True), nullable=True)
```

## Шаг 3 — Обновить схемы Pydantic

Файл: `app/schemas/country.py`

Добавить поля в схему CountryDetail (или как она называется в проекте):
```python
safety_level: str | None = None
safety_note: str | None = None
safety_source: str | None = None
cost_level: str | None = None
cost_per_day_usd: int | None = None
```

## Шаг 4 — Скрипт заполнения данных

Создай файл `scripts/seed_safety_cost.py`.

Напиши код заполнения полей случайнами значениями 

Скрипт должен:
1. Загрузить все страны из БД
2. Для каждой страны заполнить случайным значением (safe/unsafe/dangerous) поле safety_level
3. Для каждой страны заполнить случайным значением (low/medium/high) поле cost_level
4. Установить safety_updated_at и cost_updated_at = datetime.now(timezone.utc)
5. Вывести статистику: Страна X стран: безопастность Y, стоимость Z

## Шаг 5 — Проверка
```bash
# Запустить миграцию
alembic upgrade head

# Запустить скрипт заполнения
python scripts/seed_safety_cost.py

# Проверить в БД
docker exec -it visamap_postgres psql -U visauser -d visamap -c "
SELECT iso2, name_en, safety_level, cost_level, cost_per_day_usd
FROM countries
WHERE safety_level IS NOT NULL
ORDER BY iso2
LIMIT 20;"
```

## Шаг 6 — Обновить memory-bank

После выполнения задачи обнови файлы:
- `memory-bank/architecture.md` — добавь новые поля в описание таблицы countries
- `memory-bank/todo.md` — отметь выполнение пункты как выполненные ✅

## Критерии готовности
- [x] Миграция применена успешно ✅
- [x] Модель обновлена ✅
- [x] Схемы обновлены ✅
- [x] Скрипт создан и запущен ✅
- [x] Минимум 30 стран заполнены данными ✅
- [x] GET /countries/{iso2} возвращает safety_level и cost_level ✅
- [x] memory-bank обновлён ✅