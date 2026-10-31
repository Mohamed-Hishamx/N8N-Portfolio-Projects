# GCC Jobs Pipeline

n8n workflows that pull job postings from company career APIs and store them in Postgres, with a source registry that says where each company's jobs come from.

## Where to start

- [README_GCC_Jobs_Ingest.md](README_GCC_Jobs_Ingest.md): how the ingestion workflow works.
- [README_GCC_Jobs_DB.md](README_GCC_Jobs_DB.md): database schema and queries.

## Notes

- Exported CSV and XLSX files are snapshots from a specific collection date. They are not current vacancies.
- Collection time is not the same as the original publication date of a posting.
- Import the workflow JSON, create your Postgres credentials, and run it against a test database first.
