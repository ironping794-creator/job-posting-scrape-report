---
name: job-posting-scrape-report
description: Generate or adapt job-platform crawlers, collect batches of recruitment postings, clean and normalize job data, filter by city/salary/keyword, and produce structured spreadsheets plus job-distribution reports. Use when Codex is asked to turn a recruitment data collection workflow into a repeatable pipeline, scrape job posts, analyze campus/intern/full-time listings, or document a job-data workflow with evidence-bound claims.
---

# Job Posting Scrape Report

## Core Workflow

Use this skill to deliver a verified recruitment-data pipeline, not just a crawler snippet.

1. Resolve the data source and legal/technical constraints.
   - Prefer official/public APIs, exported CSV/XLSX files, or pages the user can access without bypassing controls.
   - Do not collect private personal data beyond what is necessary for job-search analysis.
   - Record source URLs, query parameters, capture time, and platform limitations.
2. Build or adapt the collector.
   - Start with a small probe: one city, one keyword, one page.
   - Confirm fields before scaling: title, company, city, salary, job type, requirements, publish time, detail URL, source platform.
   - Add polite paging, retry, de-duplication by URL/title/company/city, and progress logging.
3. Clean and normalize the data.
   - Normalize city names, salary text, date text, job type, and keyword labels.
   - Preserve raw columns when useful; add normalized columns instead of destroying evidence.
   - Use `scripts/clean_filter_jobs.py` when the input is CSV and the required columns are present or can be mapped.
4. Filter and rank.
   - Support filters for city, salary floor/ceiling, keyword include/exclude, company, job type, and publish recency.
   - Keep an unfiltered cleaned dataset and a filtered shortlist.
5. Report the result.
   - Produce a structured table (`.xlsx` preferred when the user needs clickable links or filtering).
   - Produce a Markdown statistics report using `references/report-template.md`.
   - Consider an interactive HTML dashboard when comparison, drill-down, filtering, or presentation would materially help.

## Data Contract

Target columns:

- `title`
- `company`
- `city`
- `salary`
- `job_type`
- `requirements`
- `publish_time`
- `detail_url`
- `source`

Recommended derived columns:

- `salary_min`
- `salary_max`
- `salary_unit`
- `matched_keywords`
- `is_target_city`
- `is_salary_known`
- `dedupe_key`

When source fields differ, create a small mapping table in the report so the transformation is auditable.

## Script Usage

For CSV input:

```bash
python scripts/clean_filter_jobs.py input.csv --out-dir outputs/jobs --cities "上海,北京,深圳" --keywords "AI,大模型,数据分析" --salary-min 8000
```

The script writes:

- `cleaned_jobs.csv`
- `filtered_jobs.csv`
- `job_distribution.md`

Patch the script for platform-specific salary formats or source schemas instead of adding ad hoc notebook code.

## Reporting Standards

Use concrete numbers:

- total raw rows
- total cleaned rows
- duplicate rows removed
- filtered rows retained
- top cities
- salary distribution
- keyword distribution
- source/platform caveats

Keep claims evidence-bound. If counts, timing, source coverage, or automation scope were not verified in the current run, mark them as user-provided, historical, or unverified instead of turning them into factual claims.
