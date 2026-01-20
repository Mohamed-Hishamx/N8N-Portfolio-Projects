# GCC Job Pipeline (Database-Driven) - n8n + Postgres + Power BI

A database-driven job ingestion system that:

- Reads **job sources** (ATS platform + API endpoint) from Postgres.
- Validates endpoints on a schedule and updates `job_sources.status` (`active` / `broken`).
- Fetches jobs from multiple ATS providers (Greenhouse, Lever, SmartRecruiters, Workable, etc).
- Normalizes all job payloads into a single `jobs` table.
- **Upserts** jobs using `job_url` as the natural key, so re-runs do not create duplicates.

This repo is meant to be portfolio-ready: you can run it locally, schedule it, and connect it to Power BI.

## Architecture

1. `companies` holds the GCC company list + metadata.
2. `job_sources` holds one row per source (company + ATS platform + API endpoint) and operational fields like `status`, `last_validated`, and `consecutive_failures`.
3. `jobs` holds normalized job postings.

## n8n workflow (high level)

This workflow implements three logical stages:

- **Source validation**: HTTP-check each `api_endpoint` and update `job_sources.status` + `error_message`.
- **Ingestion**: for each valid source, fetch, extract, normalize, then write to `jobs`.
- **Stats update**: update `job_sources.total_jobs_fetched` and timestamps so you can measure source quality.

## Database setup

### Required constraint for idempotent ingestion

Create a unique index on `job_url` so you can safely upsert:

```sql
CREATE UNIQUE INDEX IF NOT EXISTS idx_jobs_job_url
ON public.jobs (job_url);
```

### Recommended constraints

```sql
-- one row per (company_id, ats_platform)
CREATE UNIQUE INDEX IF NOT EXISTS idx_job_sources_company_platform
ON public.job_sources (company_id, ats_platform);

-- optional: enforce endpoint uniqueness when not null
CREATE UNIQUE INDEX IF NOT EXISTS idx_job_sources_api_endpoint_not_null
ON public.job_sources (api_endpoint)
WHERE api_endpoint IS NOT NULL;
```

## Running

1. Create your Postgres database and tables.
2. Import the workflow JSON into n8n (Workflows -> Import).
3. Configure Postgres credentials in n8n.
4. Run once manually to confirm:
   - sources validate
   - jobs are inserted/updated
5. Enable the schedule triggers.

## Monitoring SQL

How many sources are usable?

```sql
SELECT
  COUNT(*) AS total,
  COUNT(api_endpoint) AS with_endpoint,
  COUNT(*) FILTER (
    WHERE status='active'
      AND api_endpoint IS NOT NULL
      AND ats_platform <> 'unknown'
  ) AS active_usable
FROM job_sources;
```

Jobs per source (and jobs collected today):

```sql
SELECT
  source_id,
  COUNT(*) AS jobs_total,
  COUNT(*) FILTER (WHERE collected_at::date = CURRENT_DATE) AS jobs_today
FROM jobs
GROUP BY source_id
ORDER BY jobs_today DESC, jobs_total DESC;
```

## Common errors

- **"no unique or exclusion constraint matching the ON CONFLICT specification"**
  - You are trying to upsert without a unique index on the match column.
  - Fix: create `idx_jobs_job_url`.

- **"duplicate key value violates unique constraint job_sources_company_id_ats_platform_key"**
  - You inserted a duplicate (company_id, ats_platform).
  - Fix: use Insert or Update for `job_sources` and match on `company_id, ats_platform`.

- **"Column to match on not found in input item" (n8n Postgres Insert/Update)**
  - The incoming item JSON does not include the matching fields.
  - Fix: ensure the upstream node outputs `company_id` (or whatever you match on).

## Power BI dashboard ideas

- Jobs over time (daily trend) by country
- Jobs by company and ATS source
- Category/department distribution
- "New jobs today" KPI
- Source quality: active vs broken sources, jobs per source

## Disclaimer

This is a demo/portfolio project. Respect provider Terms of Service and rate limit HTTP requests.

## Database setup

### Recommended constraints / indexes

These make the pipeline idempotent and avoid ON CONFLICT errors:

```sql
-- jobs: prevent duplicates across runs
CREATE UNIQUE INDEX IF NOT EXISTS idx_jobs_job_url
ON public.jobs (job_url);

-- job_sources: one row per (company_id, ats_platform)
CREATE UNIQUE INDEX IF NOT EXISTS idx_job_sources_company_platform
ON public.job_sources (company_id, ats_platform);

-- optional: keep api_endpoint unique when not null
CREATE UNIQUE INDEX IF NOT EXISTS idx_job_sources_api_endpoint_not_null
ON public.job_sources (api_endpoint)
WHERE api_endpoint IS NOT NULL;
```

### Minimum table columns

This workflow expects these columns to exist:

- `companies`: `id`, `name`, `country`, `industry`, `website`, `careers_url`
- `job_sources`: `id`, `company_id`, `ats_platform`, `api_endpoint`, `confidence_score`, `status`, `last_validated`, `last_successful_fetch`, `consecutive_failures`, `total_jobs_fetched`, `error_message`, `created_at`, `updated_at`
- `jobs`: `id`, `title`, `company`, `location`, `country`, `salary_min`, `salary_max`, `contract_type`, `category`, `source`, `job_url`, `posted_at`, `collected_at`, `source_id`, `is_active`, `created_at`, `updated_at`

## How updates work (no duplicates)

The jobs write step should be configured as **Insert or Update** and match on `job_url`.

- If the job URL already exists, the row gets updated (e.g., `is_active`, `updated_at`, etc).
- If the job URL is new, it inserts a new row.

## Running it

1) Create Postgres DB + tables (or run your schema migration).
2) Import the workflow JSON into n8n.
3) Configure Postgres credentials in n8n.
4) Run the workflow once manually.
5) Enable the schedule trigger.

## Monitoring queries

### Source health

```sql
SELECT status, COUNT(*)
FROM job_sources
GROUP BY status
ORDER BY COUNT(*) DESC;
```

### Sources with endpoints

```sql
SELECT ats_platform, COUNT(*)
FROM job_sources
WHERE api_endpoint IS NOT NULL
GROUP BY ats_platform
ORDER BY COUNT(*) DESC;
```

### Jobs by source

```sql
SELECT
  source_id,
  COUNT(*) AS jobs_total,
  COUNT(*) FILTER (WHERE collected_at::date = CURRENT_DATE) AS jobs_today
FROM jobs
GROUP BY source_id
ORDER BY jobs_today DESC, jobs_total DESC;
```

## Troubleshooting

- **ERROR: no unique or exclusion constraint matching the ON CONFLICT specification**
  - You are using UPSERT on a non-unique column. Add a unique index (most commonly `jobs(job_url)`).

- **duplicate key value violates unique constraint job_sources_company_id_ats_platform_key**
  - You are inserting duplicates into `job_sources`. Switch that node to **Insert or Update** and match on `(company_id, ats_platform)`.

- **Column to match on not found in input item**
  - In n8n, your input JSON must include the matching fields. Example: if matching on `company_id`, the item must contain `company_id`.

## Database setup

### Recommended constraints / indexes

These make the pipeline idempotent and avoid `ON CONFLICT` errors:

```sql
-- jobs: prevent duplicates across runs
CREATE UNIQUE INDEX IF NOT EXISTS idx_jobs_job_url
ON public.jobs (job_url);

-- job_sources: one row per (company_id, ats_platform)
CREATE UNIQUE INDEX IF NOT EXISTS idx_job_sources_company_platform
ON public.job_sources (company_id, ats_platform);

-- optional: keep api_endpoint unique when it exists
CREATE UNIQUE INDEX IF NOT EXISTS idx_job_sources_api_endpoint_not_null
ON public.job_sources (api_endpoint)
WHERE api_endpoint IS NOT NULL;
```

### Minimal table expectations

The ingestion workflow expects these fields (yours may have more):

- `companies(id, name, country, industry, website, careers_url, ...)`
- `job_sources(id, company_id, ats_platform, api_endpoint, status, last_validated, last_successful_fetch, consecutive_failures, total_jobs_fetched, error_message, created_at, updated_at)`
- `jobs(id, title, company, location, country, salary_min, salary_max, contract_type, category, source, job_url, posted_at, collected_at, source_id, is_active, created_at, updated_at)`

## Running the project

1) Create your Postgres database and tables.
2) Import the n8n workflow JSON.
3) Configure the Postgres credentials in n8n.
4) Run once manually to validate sources and populate `jobs`.
5) Enable schedules (Cron) for ongoing ingestion.

## Monitoring queries

How many sources are usable right now?

```sql
SELECT
  COUNT(*) AS total,
  COUNT(api_endpoint) AS with_endpoint,
  COUNT(*) FILTER (
    WHERE status='active'
      AND api_endpoint IS NOT NULL
      AND ats_platform <> 'unknown'
  ) AS active_usable
FROM job_sources;
```

Jobs by source (and jobs collected today):

```sql
SELECT
  source_id,
  COUNT(*) AS jobs_total,
  COUNT(*) FILTER (WHERE collected_at::date = CURRENT_DATE) AS jobs_today
FROM jobs
GROUP BY source_id
ORDER BY jobs_today DESC, jobs_total DESC;
```

## Troubleshooting

- **`no unique or exclusion constraint matching the ON CONFLICT specification`**
  - You are upserting on a column without a UNIQUE constraint.
  - Fix: create the unique index (usually `idx_jobs_job_url`).

- **`duplicate key value violates unique constraint job_sources_company_id_ats_platform_key`**
  - You are inserting duplicates into `job_sources`.
  - Fix: change the node to **Insert or Update** and match on `(company_id, ats_platform)`.

- **`Column to match on not found in input item` (n8n Postgres node)**
  - Your incoming JSON item does not contain the matching column.
  - Fix: ensure the item has `company_id` / `ats_platform` / or whatever you selected as "Columns to match".

## Power BI

Connect Power BI to Postgres (or export `jobs` to CSV) and build:

- Jobs over time (daily/weekly)
- Jobs by country and company
- Categories and contract types
- Source quality (active vs broken sources; jobs fetched per source)

Tip: create a Date table and relate it to `jobs[collected_at]`.

## Database setup

### Recommended constraints / indexes

These make the pipeline idempotent and avoid `ON CONFLICT` errors:

```sql
-- jobs: prevent duplicates across runs
CREATE UNIQUE INDEX IF NOT EXISTS idx_jobs_job_url
ON public.jobs (job_url);

-- job_sources: one row per (company_id, ats_platform)
CREATE UNIQUE INDEX IF NOT EXISTS idx_job_sources_company_platform
ON public.job_sources (company_id, ats_platform);

-- optional: keep api_endpoint unique when it exists
CREATE UNIQUE INDEX IF NOT EXISTS idx_job_sources_api_endpoint_not_null
ON public.job_sources (api_endpoint)
WHERE api_endpoint IS NOT NULL;
```

### Minimal table expectations

The ingestion workflow expects these columns (names can vary, but keep mapping consistent):

- `companies`: `id`, `name`, `country`, `industry`, `website`, `careers_url`
- `job_sources`: `id`, `company_id`, `ats_platform`, `api_endpoint`, `status`, `last_validated`, `last_successful_fetch`, `consecutive_failures`, `total_jobs_fetched`, `error_message`
- `jobs`: `id`, `title`, `company`, `location`, `country`, `salary_min`, `salary_max`, `contract_type`, `category`, `source`, `job_url`, `posted_at`, `collected_at`, `source_id`, `is_active`

## Running the pipeline

1. Create your Postgres DB and tables.
2. Import the n8n workflow JSON.
3. Configure Postgres credentials in n8n.
4. Run once manually to confirm:
   - Sources get validated.
   - Jobs land in `jobs`.
5. Turn on schedules.

## Monitoring queries

```sql
-- Sources by status
SELECT status, COUNT(*)
FROM job_sources
GROUP BY status
ORDER BY COUNT(*) DESC;

-- Jobs by source (today vs total)
SELECT
  source_id,
  COUNT(*) AS jobs_total,
  COUNT(*) FILTER (WHERE collected_at::date = CURRENT_DATE) AS jobs_today
FROM jobs
GROUP BY source_id
ORDER BY jobs_today DESC, jobs_total DESC;
```

## Troubleshooting

- **"no unique or exclusion constraint matching the ON CONFLICT specification"**
  - You are UPSERTing on a non-unique column. Create a unique index on what you match on (usually `jobs.job_url`).

- **"duplicate key value violates unique constraint job_sources_company_id_ats_platform_key"**
  - Use an UPSERT (Insert or Update) on `job_sources` matching `company_id, ats_platform`, or remove duplicates.

- **"Column to match on not found in input item" (n8n Postgres node)**
  - Your input JSON does not include the matching columns. Ensure the upstream node outputs (for example) `company_id` and `ats_platform` if you match on those.

## Power BI dashboard ideas

- Job volume over time (day/week) by country
- Jobs by company and industry
- Category distribution
- New jobs today KPI
- Source reliability (active vs broken sources)

Tip: build a Calendar table in Power BI and relate it to `jobs[collected_at]`.

## Disclaimer

This is a demo/portfolio project. Respect each provider's Terms of Service and rate limit your requests.
