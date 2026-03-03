# Patent Retriever (BigQuery)

A focused OpenClaw skill for patent retrieval using **Google Patent BigQuery**.

This repository is designed for:
- standalone GitHub hosting
- ClawHub import from GitHub URL
- structured retrieval output for downstream analysis/drafting

## GitHub Repository

- Repository: `https://github.com/yangyuwen-bri/patent-retriever-bigquery`
- Recommended import source for ClawHub: this repository root (contains `SKILL.md` at root)

## What This Skill Does

- Runs seed patent retrieval by topic/keywords
- Builds a scanner-style `concept_scan` and `query_plan`
- Executes multi-round retrieval from `patents-public-data.patents.publications`
- Enforces minimum retrieval volume (default `>= 20`)
- Outputs structured JSON for downstream pipelines

## What This Skill Does NOT Do

- No legal advice
- No patentability determination
- No filing strategy recommendation
- No final disclosure/claims drafting in this repo

## Repository Structure

```text
patent-retriever-bigquery/
├── SKILL.md
├── README.md
├── requirements.txt
├── .env.example
├── .gitignore
├── scripts/
│   ├── config.py
│   ├── patent_search.py
│   ├── build_query_plan.py
│   ├── patent_search_plan.py
│   └── schema_check.py
├── schemas/
│   ├── concept_scan.schema.json
│   ├── query_plan.schema.json
│   └── retriever_result.schema.json
├── references/
│   └── methodology.md
└── examples/
    └── quickstart.md
```

## Prerequisites

Required environment variables:

- `GOOGLE_APPLICATION_CREDENTIALS` (absolute path to service account JSON)
- `GOOGLE_CLOUD_PROJECT` (your GCP project id)

Install dependencies:

```bash
python3 -m pip install -r requirements.txt
```

## Quick Start

### 1) Seed retrieval

```bash
python3 scripts/patent_search.py \
  --keywords "ai sentiment analysis" \
  --limit 80 \
  --output results/seed_raw.json
```

### 2) Build concept scan + query plan

```bash
python3 scripts/build_query_plan.py \
  --topic "舆情+AI" \
  --keywords "public opinion ai sentiment" \
  --task-id "demo_001" \
  --seed-raw results/seed_raw.json \
  --concept-output results/concept_scan.json \
  --plan-output results/query_plan.json
```

### 3) Execute planned retrieval

```bash
python3 scripts/patent_search_plan.py \
  --plan results/query_plan.json \
  --output-raw results/retriever_raw.json \
  --output-retriever results/retriever_result.json \
  --min-results 20
```

### 4) Validate outputs

```bash
python3 scripts/schema_check.py --input results/concept_scan.json --schema schemas/concept_scan.schema.json
python3 scripts/schema_check.py --input results/query_plan.json --schema schemas/query_plan.schema.json
python3 scripts/schema_check.py --input results/retriever_result.json --schema schemas/retriever_result.schema.json
```

## Output Files

- `results/concept_scan.json`
- `results/query_plan.json`
- `results/retriever_raw.json`
- `results/retriever_result.json`

`retriever_result.json` contract:
- `patents` count is expected to be `>= 20`
- each item should contain at least `publication_number` and `title`

## ClawHub Import (from GitHub)

1. Prepare and push your code to:
   - `https://github.com/yangyuwen-bri/patent-retriever-bigquery`
2. In ClawHub website, choose import from GitHub URL
3. Paste repo URL:
   - `https://github.com/yangyuwen-bri/patent-retriever-bigquery`
4. Confirm import root contains `SKILL.md`
5. Complete metadata (name/slug/tags/version), then publish

## Notes on Cost and Quota

- Retrieval uses BigQuery queries and may incur cost depending on your project billing settings.
- If query fails due to project/API/quota permissions, check:
  - BigQuery API enabled
  - service account permissions
  - billing/quota status

## Security Notes

- Use a dedicated service account with least privilege.
- Prefer read-only BigQuery access scoped to Google Patents public dataset/table.
- Do not commit credential JSON files or secrets into this repository.
