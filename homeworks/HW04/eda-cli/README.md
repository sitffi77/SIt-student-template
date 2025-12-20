## CLI

### Краткий обзор датасета

```bash
uv run eda-cli overview data/example.csv
```

Дополнительные параметры:
- `--sep` — разделитель колонок (по умолчанию `,`)
- `--encoding` — кодировка файла (по умолчанию `utf-8`)

Пример для файла с точкой с запятой и windows-1251:

```bash
uv run eda-cli overview my_data.csv --sep ";" --encoding cp1251
```

### Полный EDA-отчёт

```bash
uv run eda-cli report data/example.csv --out-dir reports
```

#### Основные параметры команды `report`

| Параметр                  | Описание                                                                 | Значение по умолчанию | Пример                              |
|---------------------------|--------------------------------------------------------------------------|-----------------------|-------------------------------------|
| `csv_path`                | Путь к CSV-файлу                                                         | —                     | `data/example.csv`                  |
| `--out-dir`               | Каталог для сохранения результатов отчёта                                | `report`              | `--out-dir my_reports`              |
| `--title`                 | Заголовок отчёта в Markdown                                              | `EDA-отчёт`           | `--title "Анализ данных пользователей"` |
| `--top-k-categories`      | Количество топ-значений для категориальных признаков                     | `5`                   | `--top-k-categories 10`             |
| `--sep`                   | Разделитель колонок в CSV                                                | `,`                   | `--sep ";"`                         |
| `--encoding`              | Кодировка CSV-файла                                                      | `utf-8`               | `--encoding cp1251`                 |
| `--max-rows`              | Максимальное количество строк для чтения (для больших файлов)            | все строки            | `--max-rows 10000`                  |

#### Что генерируется в `--out-dir`

После выполнения команды в указанном каталоге появятся:

- `report.md` — основной отчёт в формате Markdown
- `summary.csv` — общая сводка по колонкам (типы, пропуски, уникальные значения и т.д.)
- `missing.csv` — детализация пропусков по колонкам
- `correlation.csv` — корреляционная матрица (только для числовых колонок)
- `top_categories/<column>.csv` — топ-значения для каждого категориального признака
- `hist_<column>.png` — гистограммы распределения для числовых колонок
- `missing_matrix.png` — визуализация матрицы пропусков
- `correlation_heatmap.png` — тепловая карта корреляций (если есть числовые признаки)

Пример полного вызова:

```bash
uv run eda-cli report my_data.csv \
  --out-dir my_report \
  --title "Анализ своих данных" \
  --top-k-categories 7 \
  --sep ";" \
  --encoding cp1251
```

## HTTP-сервис (FastAPI)

### Запуск

```bash
uv run uvicorn eda_cli.api:app --reload --port 8000
```

Интерактивная документация и тестирование запросов:  
http://127.0.0.1:8000/docs

### Эндпоинты

| Метод | Путь                        | Описание                                                                 |
|-------|-----------------------------|--------------------------------------------------------------------------|
| GET   | `/health`                   | Health-check сервиса                                                     |
| POST  | `/quality`                  | Оценка качества по агрегированным метрикам (принимает JSON)              |
| POST  | `/quality-from-csv`         | Оценка качества по загруженному CSV-файлу (form-data)                    |
| POST  | `/quality-flags-from-csv`   | ✅ HW04 — полный набор булевых флагов качества (включая все эвристики)   |

### Примеры запросов

1. Health-check

```bash
curl http://127.0.0.1:8000/health
```

2. Оценка по агрегированным признакам (JSON)

```bash
curl -X POST http://127.0.0.1:8000/quality \
  -H "Content-Type: application/json" \
  -d '{
    "n_rows": 5000,
    "n_cols": 12,
    "max_missing_share": 0.05,
    "numeric_cols": 8,
    "categorical_cols": 4
  }'
```

3. Оценка качества из CSV-файла

```bash
curl -X POST http://127.0.0.1:8000/quality-from-csv \
  -F "file=@data/example.csv"
```

4. Полные флаги качества из CSV (HW04)

```bash
curl -X POST http://127.0.0.1:8000/quality-flags-from-csv \
  -F "file=@data/example.csv"
```

Ответ содержит все булевы флаги, например: `has_constant_columns`, `has_high_cardinality_categoricals`, `has_duplicates` и др.

## Тесты

```bash
uv run pytest -q
```

Ожидается: 4 теста пройдено (PASSED).

## Что нового в HW04

1. Добавлен FastAPI-сервер с эндпоинтами `/health`, `/quality`, `/quality-from-csv`.
2. Новый эндпоинт `/quality-flags-from-csv` — возвращает полный набор флагов качества датасета.
3. Добавлены зависимости: `fastapi`, `uvicorn[standard]`, `python-multipart`.
4. Интерактивная документация Swagger/ReDoc по адресу `/docs`.
