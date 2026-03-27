# Analytics Upload MVP Integration (Private-by-Default)

## 1) Feature overview

Analytics Upload MVP allows a user to upload a legal document for **private analysis** (extraction + AI-assisted interpretation) without adding that document to the public legal corpus.

### What it is
- A separate analytics workflow for uploaded legal files.
- Designed for case preparation, internal review, and exploratory legal analysis.
- Produces structured results that frontend, backend, and runtime can share.

### Why private by default
- Uploaded files may contain sensitive strategy, personal data, or non-public submissions.
- Teams need analysis support without accidental publication.
- Privacy-first behavior reduces legal and operational risk.

### How it differs from public corpus ingestion
- **Analytics Upload**: user-provided, private, run-scoped results.
- **Public ingestion**: curated/approved legal sources for searchable public corpus.
- No automatic linking from analytics uploads into public `presude` / `zakoni` flows.

---

## 2) Repo responsibilities

### `ai.jpis.ba-front`
- Provide upload UI for legal documents.
- Call `POST /jpis/analytics/upload` and store returned `{ id, status }`.
- Poll or fetch `GET /jpis/analytics/:id` for lifecycle updates.
- Render analysis result in private user/workspace context.
- Keep analytics views separate from public corpus navigation/search.

### `ai.jpis.ba-backend`
- Receive upload, create analytics run, set initial status (`queued`).
- Orchestrate extraction/analysis steps and status transitions.
- Expose contract-stable responses for both upload and result endpoints.
- Enforce private-by-default boundary in API and storage references.
- Avoid exposing local/internal file paths in public-facing payloads.

### `jpis-ai-runtime`
- Execute extraction and analysis workloads when dispatched.
- Return structured artifacts compatible with shared result shape.
- Report runtime stage progress so backend can map to shared statuses.
- Preserve compatibility with `document_kind` vocabulary and candidate fields.

---

## 3) Shared API contract

### Required authn/authz contract (MVP)

This is a required part of the API contract and **must not** be treated as optional:
- Every analytics endpoint requires an authenticated caller identity.
- Every analytics run is bound to both:
  - `owner_user_id` (the creator), and
  - `workspace_id` (the tenant/workspace context at creation time).
- Access to `GET /jpis/analytics/:id` is allowed only when the caller belongs to the same `workspace_id`.
- Within that workspace, access is further restricted to:
  - the `owner_user_id`, or
  - a workspace member with an explicit analytics-read permission (e.g., workspace admin/reviewer role).
- If the run exists but caller is not authorized, return `404` (or equivalent non-disclosing response) to avoid leaking run existence across tenants.
- ID values must be treated as opaque identifiers; possession of `:id` alone never grants read access.

## POST `/jpis/analytics/upload`
Upload a legal document for private analytics processing.

### Response
```json
{
  "id": "string",
  "status": "queued|extracting|analyzing|completed|failed"
}
```

## GET `/jpis/analytics/:id`
Retrieve the analytics run state and result payload.

### Response
- Returns the shared result shape described below.

---

## 4) Shared result shape

All repos should converge toward this response JSON:

```json
{
  "id": "string",
  "status": "queued|extracting|analyzing|completed|failed",
  "file_name": "string",
  "mime_type": "string",
  "uploaded_at": "ISO datetime string",
  "document_kind": "court_decision|law|submission|unknown_legal_document",
  "extraction_quality": "string | null",
  "summary": "string | null",
  "key_points": ["string"],
  "referenced_laws": ["string"],
  "referenced_articles": ["string"],
  "candidate_related_laws": ["string"],
  "candidate_related_decisions": ["string"],
  "structured_sections": [
    {
      "key": "string",
      "label": "string | null",
      "content": "string"
    }
  ],
  "warnings": ["string"],
  "confidence_notes": ["string"],
  "private_by_default": true
}
```

### Compatibility notes
- If a repo cannot fill all fields yet, return the minimum compatible subset with additive evolution.
- Prefer agreed names above over custom field renaming.

---

## 5) Status lifecycle

Standard lifecycle:
- `queued`: upload accepted, waiting for processing.
- `extracting`: text/structure extraction in progress.
- `analyzing`: AI/legal analysis step in progress.
- `completed`: processing finished successfully.
- `failed`: processing failed (include useful warning/error context when possible).

Implementation guidance:
- Backend is source of truth for external status.
- Runtime may use internal stages, but map externally to this shared vocabulary.

---

## 6) Privacy boundary

MVP privacy rules:
- Uploads remain private by default.
- Analytics results remain private by default.
- No automatic insertion into public corpus ingestion pipelines.
- No automatic indexing into public search.
- No automatic exposure via public `presude/zakoni` endpoints.
- Reviewed promotion to public corpus may exist later as a separate, explicit workflow.

---

## 7) Limitations in MVP

Expected constraints:
- Extraction quality varies by source quality (scan quality, formatting, language/style variance).
- OCR support may be incomplete or inconsistent for some scanned PDFs/images.
- Candidate relations (`candidate_related_*`) can be heuristic and non-authoritative.
- `document_kind` classification may be uncertain; fallback to `unknown_legal_document` when needed.
- Referenced law/article extraction may miss edge citations or complex legal drafting patterns.

---

## 8) Next recommended steps

1. Add private upload history / workspace run list with filters.
2. Improve citation extraction for laws and article-level references.
3. Strengthen law/article entity linking against canonical identifiers.
4. Add "compare uploaded document vs known corpus" assistance.
5. Design and implement reviewed promotion workflow (private -> public) with explicit approval trail.

---

## Open integration questions

1. Retention policy: how long are private uploads/artifacts stored by default?
2. Error model: should `failed` include standardized machine-readable error codes?
3. Polling vs push: should frontend rely on polling only, or add SSE/WebSocket updates later?
4. Promotion governance: which role(s) can approve private-to-public promotion?
