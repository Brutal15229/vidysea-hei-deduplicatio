# Vidysea HEI Deduplication & Golden Record Builder

This repository contains a repeatable Python pipeline for the Vidysea data engineering take-home challenge.

It reads the provided MongoDB `mongodump --gzip` files from `global_university_db/`, filters United States and Japan records, deduplicates universities into golden records, maps and deduplicates programs, and writes the requested JSONL outputs and reports.

## Setup

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

## Run

```powershell
python -m vidysea_dedupe --dump-dir global_university_db --output-dir outputs
```

Expected outputs:

- `outputs/canonical_universities_usa.jsonl`
- `outputs/canonical_universities_japan.jsonl`
- `outputs/canonical_programs_usa.jsonl`
- `outputs/canonical_programs_japan.jsonl`
- `outputs/manual_review_universities.jsonl`
- `outputs/manual_review_programs.jsonl`
- `outputs/data_quality_report.md`
- `outputs/run_log.json`

## Notes

The workspace includes `university_schema_v8.json` and `program_schema_v4.json`. The pipeline emits records shaped to those schema versions and preserves source provenance for every canonical university and program.
