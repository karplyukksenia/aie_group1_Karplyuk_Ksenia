# S03 – eda_cli: мини-EDA для CSV

Небольшое CLI-приложение для базового анализа CSV-файлов.
Используется в рамках Семинара 03 курса «Инженерия ИИ».

## Требования

- Python 3.11+
- [uv](https://docs.astral.sh/uv/) установлен в систему

## Инициализация проекта

В корне проекта (S03):

```bash
uv sync
```

Эта команда:

- создаст виртуальное окружение `.venv`;
- установит зависимости из `pyproject.toml`;
- установит сам проект `eda-cli` в окружение.

## Запуск CLI

### Краткий обзор

```bash
uv run eda-cli overview data/example.csv
```

Параметры:

- `--sep` – разделитель (по умолчанию `,`);
- `--encoding` – кодировка (по умолчанию `utf-8`).
- `--title' - название отчета (по умолчанию "EDA-отчёт")
Пример использования: 
```bash
uv run eda-cli report .\data\example.csv --out-dir reports_example --title "TITLE"
```
- `--top-k-categories` - ограничение по количеству "топ" категорий (по умолчанию 10)
Пример использования: 
```bash
uv run eda-cli report .\data\example.csv --out-dir reports_example ----top-k-categories 5
```


### Полный EDA-отчёт

```bash
uv run eda-cli report data/example.csv --out-dir reports
```

В результате в каталоге `reports/` появятся:

- `report.md` – основной отчёт в Markdown;
- `summary.csv` – таблица по колонкам;
- `missing.csv` – пропуски по колонкам;
- `correlation.csv` – корреляционная матрица (если есть числовые признаки);
- `top_categories/*.csv` – top-k категорий по строковым признакам;
- `hist_*.png` – гистограммы числовых колонок;
- `missing_matrix.png` – визуализация пропусков;
- `correlation_heatmap.png` – тепловая карта корреляций.

В файле report.md добавлены новые показатели:
- Наличие колонок, с одинаковыми значениями (True/False);
- Наличие категориальных признаков с большой уникальностью (True/False);
- Наличие колонок с большой долей нулей (True/False).


## Тесты

```bash
uv run pytest -q
```
