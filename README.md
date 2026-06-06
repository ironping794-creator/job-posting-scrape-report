# Job Posting Scrape Report

Generate or adapt job-platform crawlers, collect batches of recruitment postings, clean and normalize job data, filter by city / salary / keyword, and produce structured spreadsheets plus distribution reports.

Contains two standalone Python scripts plus a reusable Codex skill definition for building verified recruitment-data pipelines from public JSON APIs and CSV exports.

## Features

- **Paginated API collector** — walks public JSON APIs with polite delays, estimates runtime before fetching, supports interactive or scripted mode
- **CSV cleaner / filter** — normalizes city names, salary ranges, and keywords from CSV exports; deduplicates by URL + title + company + city
- **Configurable output** — cleaned CSV, filtered shortlist, Markdown distribution report, raw JSON for debugging
- **Platform neutral** — provide URL, method, payload, and JSON paths; the collector handles the rest
- **Codex skill integration** — the `SKILL.md` file teaches Codex to use this workflow end-to-end

## Repository Structure

```
job-posting-scrape-report/
  SKILL.md                    # Codex skill instructions (workflow + data contract)
  agents/openai.yaml          # OpenAI agent metadata
  scripts/
    collect_paginated_jobs.py  # Walks paginated JSON APIs with polite delays
    clean_filter_jobs.py       # Normalizes and filters CSV exports
  references/
    report-template.md         # Markdown report skeleton
```

## Quick Start

### Collect from a paginated JSON API

```bash
python scripts/collect_paginated_jobs.py \
  --url "https://example.com/api/jobs/page-list" \
  --method POST \
  --payload '{"cityId":35}' \
  --page-param page \
  --size-param size \
  --records-path data.records \
  --total-path data.total \
  --page-size 50 \
  --out-dir outputs/my-job-collection
```

When `--limit` is omitted in an interactive terminal, the script probes page 1, estimates the total runtime, and asks whether to collect `all`, `half`, or a specific number. In non-interactive runs, pass `--limit all` or `--limit 200`.

### Clean and filter a CSV export

```bash
python scripts/clean_filter_jobs.py input.csv \
  --out-dir outputs/filtered \
  --cities "上海,北京,深圳" \
  --keywords "AI,大模型,数据分析" \
  --salary-min 8000
```

The script normalizes salary text, deduplicates records, applies filters, and writes `cleaned_jobs.csv`, `filtered_jobs.csv`, and `job_distribution.md`.

## Script Reference

### `collect_paginated_jobs.py`

| Argument | Default | Description |
|---|---|---|
| `--url` | (required) | JSON API endpoint |
| `--method` | POST | HTTP method (GET / POST) |
| `--headers` | "" | JSON object or path to a JSON file |
| `--payload` | {} | JSON object or path to a JSON file |
| `--page-param` | page | Dotted path for page number in payload |
| `--size-param` | size | Dotted path for page size in payload |
| `--page-size` | 50 | Records per page |
| `--records-path` | data.records | Dotted JSON path to records list |
| `--total-path` | data.total | Dotted JSON path to total count |
| `--pages-path` | data.pages | Dotted JSON path to page count |
| `--limit` | prompt | all / half / integer |
| `--max-pages` | none | Hard page cap |
| `--delay` | 0.5 | Seconds between page requests |
| `--timeout` | 30 | Request timeout |

### `clean_filter_jobs.py`

| Argument | Default | Description |
|---|---|---|
| `input_csv` | (required) | Path to CSV file |
| `--out-dir` | outputs/jobs | Output directory |
| `--cities` | "" | Comma-separated city filter |
| `--keywords` | "" | Comma-separated keyword filter |
| `--salary-min` | none | Minimum salary threshold |

## Output Files

The collector writes into the specified `--out-dir`:

- `raw_pages.json` — full API responses
- `records.json` — extracted records
- `records.csv` — records in tabular form (when records are JSON objects)
- `summary.json` — capture metadata (source URL, timing, counts)

The CSV cleaner writes into its `--out-dir`:

- `cleaned_jobs.csv` — deduplicated, normalized rows
- `filtered_jobs.csv` — rows matching city / keyword / salary filters
- `job_distribution.md` — Markdown statistics report

## Data Contract

Target columns recognized by the CSV cleaner: `title`, `company`, `city`, `salary`, `job_type`, `requirements`, `publish_time`, `detail_url`, `source`.

Common Chinese column-name variants (岗位 / 公司 / 城市 / 薪资 / 类型 / 要求 / 发布时间) are mapped automatically.

## License

MIT
