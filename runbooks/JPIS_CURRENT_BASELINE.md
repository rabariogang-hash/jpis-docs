# JPIS Current Baseline

## Baseline Date

2026-06-16

## Repository Map

- `D:\JPISAI\ai.jpis.ba-front` - frontend application
- `D:\JPISAI\ai.jpis.ba-backend` - backend API and import endpoints
- `D:\JPISAI\jpis-ai-runtime` - document worker, parser, legal reference extraction
- `D:\JPISAI\jpis-ingestion` - ingestion scripts, dry-run/push workflows, local import evidence
- `D:\JPISAI\jpis-datasets` - canonical datasets and schemas
- `D:\JPISAI\jpis-docs` - runbooks and operational documentation
- `D:\JPISAI\data-raw` - local raw source material workspace
- `D:\JPISAI\data-processing` - local processing/review workspace

## Production URLs

- Frontend: https://ai-jpis-ba-front.vercel.app
- Backend: https://ai-jpis-ba-backend.onrender.com

## Current Laws in the System

| Canonical ID | Short title | Article count | Operational status |
| --- | --- | ---: | --- |
| `criminal_code_bih` | KZ BiH | 277 | `consolidated_latest_preview_ready` |
| `criminal_procedure_code_bih` | ZKP BiH | 452 | `baseline_segmented_clean_v4` |
| `zoo_fbih` | ZOO FBiH | 1109 | `operational_preview` |
| `zpp_fbih` | ZPP FBiH | n/a | `operational_preview` |
| `zpd_fbih` | ZPD FBiH | n/a | `operational_preview` |

## Court Decisions

- 19 `golden-core-20` court decision JSON documents are imported in the backend.
- Court decision import endpoint: `/jpis/import/court-decisions`.
- Court decision detail route supports `canonical_id`.
- Frontend `/presude` list cards no longer display `decision.summary`; detailed content is shown only after opening a decision.

## Key Commits

### jpis-datasets

- `e58a34e` Add ZKP BiH canonical law dataset
- `30abb2d` Update court decision schema for structured dataset
- `3974489` Add structured golden court decision dataset
- `aab59fb` Add core court decisions golden set

### jpis-docs

- `1de2ed6` Add court decisions ingestion workflow runbook
- `b38ad21` Add legal acts ingestion runbook

### jpis-ingestion

- `d6a4890` Skip non-decision JSON files during court decision ingest

### jpis-ai-runtime

- `18b8cd7` Canonicalize court decision legal reference titles

### ai.jpis.ba-front

- `672d3a7` Hide court decision summaries from list cards

### ai.jpis.ba-backend

- `6399397` Merge pull request #28

## Current Repository Status

- `ai.jpis.ba-front`: clean/synced; has local stashes for previous frontend work.
- `ai.jpis.ba-backend`: clean/synced.
- `jpis-ai-runtime`: clean/synced; has one local stash for a stale doc-worker health payload.
- `jpis-ingestion`: synced but locally dirty because of logs, outputs, helper scripts, and import evidence.
- `jpis-datasets`: clean/synced.
- `jpis-docs`: clean/synced.

## Operating Rules

- Never run `git reset --hard`.
- Never run `git clean`.
- Never run `git add .`.
- Never run an import without a clean dry-run first.
- Never write to production API or DB without explicit approval.
- Never print `JPIS_IMPORT_API_KEY` or any equivalent secret value.
- Do not delete `jpis-ingestion` outputs or logs without review.

## Next Legal Priorities

- `Zakon o upravnim sporovima Republike Srpske`
- `Zakon o opstem upravnom postupku RS/FBiH`
- `Zakon o stvarnim pravima RS/FBiH`
- Brcko procedural laws, if the next workstream continues focusing on Brcko court decisions

## Status Semantics

All laws currently visible in the backend/frontend are considered complete for their marked operational level. Statuses such as `operational_preview`, `baseline_segmented_clean_v4`, and `consolidated_latest_preview_ready` describe the legal and verification processing level. They do not mean the law is unfinished for the currently approved system baseline.
