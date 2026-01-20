# GCC Jobs Ingest (API List) - n8n + Postgres

This is the lightweight version of the GCC job pipeline: instead of maintaining a `job_sources` registry in the database, you keep a curated list of ATS API endpoints and ingest jobs into Postgres.

Use this when you want a simple, portfolio-friendly ingest that you can run on a schedule.

## What it does

- Builds a list of sources (company + country + ATS platform + API endpoint)
- Fetches jobs per source via HTTP
- Extracts jobs from provider-specific payloads (Greenhouse/Lever/SmartRecruiters/Workable)
- Normalizes fields into one schema
- Inserts or updates into Postgres `jobs`

## Minimal database requirements

The workflow writes into a `jobs` table with columns like:

`id, title, company, location, country, salary_min, salary_max, contract_type, category, source, job_url, posted_at, collected_at, source_id, is_active, created_at, updated_at`

Recommended:

```sql
CREATE UNIQUE INDEX IF NOT EXISTS idx_jobs_job_url
ON public.jobs (job_url);
```

## Run it

1. Create your Postgres database and the `jobs` table.
2. Import the n8n workflow JSON (Workflows -> Import from File).
3. Configure Postgres credentials in n8n.
4. Run once manually, then enable the schedule.

## Notes

- To stay idempotent across runs, use **Insert or Update** (UPSERT) in the Postgres node and match on `job_url`.
- Add rate limiting between HTTP requests to avoid getting blocked.

## Disclaimer

This is a demo/portfolio project. Respect rate limits and each provider's Terms of Service.
