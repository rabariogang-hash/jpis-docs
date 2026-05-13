# JPIS legal acts ingestion runbook

This runbook describes the controlled workflow for importing laws and other legal acts into JPIS.

It is based on the successful ZOO FBiH import where the final database state was:

- law canonical id: `zoo_fbih`
- expected article count: `1109`
- confirmed DB article count: `1109`
- important verified articles: `103`, `104`, `1108`
- final repair mode for the last 10 articles: `options.index=false` and `options.create_chunks=false`

Do not treat a large legal act import as a single blind push. Always extract, audit, build a controlled preview dataset, run a dry-run, import in controlled mode, and verify counts through the backend API.

---

## 1. Repository map

Typical local layout used during development:

```text
D:\JPISAI\ai.jpis.ba-backend      # Medusa v2 backend, Render service
D:\JPISAI\ai.jpis.ba-front        # Next.js frontend, Vercel service
D:\JPISAI\jpis-ingestion          # TypeScript ingestion pipelines
D:\JPISAI\jpis-datasets           # versioned datasets when ready
D:\JPISAI\jpis-docs               # architecture docs and runbooks
D:\JPISAI\data-raw                # local raw source files, not necessarily committed
D:\JPISAI\data-processing         # local extraction/review/build artifacts
```

Important live services:

```text
Backend:  https://ai-jpis-ba-backend.onrender.com
Frontend: https://ai-jpis-ba-front.vercel.app
```

Important ingestion scripts:

```text
jpis-ingestion/package.json
  npm run dev:law       -> tsx scripts/ingest-laws.ts
  npm run dev:cases     -> tsx scripts/ingest-cases.ts
  npm run dev:literature-> tsx scripts/ingest-literature.ts
```

---

## 2. Source selection rules

For every law/legal act, record:

- canonical law id, e.g. `zoo_fbih`
- title and short title
- jurisdiction, e.g. `BiH`, `FBiH`, `RS`, `BD BiH`
- official gazette references
- source URL
- source type: official consolidated text, official PDF, annotated PDF, manually reviewed operational preview
- review status

Preferred source order:

1. Official parliament/government gazette consolidated source when available.
2. Official gazette PDF/source document.
3. Trusted legal portal source, only if source metadata marks it as non-official or working text.
4. Annotated/commented PDF only as `operational_preview`, with manual review required.

Never hide the source quality. If the source is an annotated PDF, set metadata such as:

```json
{
  "source_kind": "autorski_precisceni_tekst",
  "source_status": "working_text_manual_review",
  "manual_review_required": true
}
```

---

## 3. Raw source download pattern

Example from ZOO FBiH/RS annotated PDF:

```powershell
New-Item -ItemType Directory -Force "D:\JPISAI\data-raw\legal-acts\bih\zoo_bih" | Out-Null

Invoke-WebRequest `
  -Uri "https://udofbih.ba/wp-content/uploads/2018/01/1.-Zakon-o-obligacionim-odnosima-FBiH-RS.pdf" `
  -OutFile "D:\JPISAI\data-raw\legal-acts\bih\zoo_bih\zoo_fbih_rs.pdf"

Get-Item "D:\JPISAI\data-raw\legal-acts\bih\zoo_bih\zoo_fbih_rs.pdf" |
  Select-Object FullName, Length, LastWriteTime
```

Keep raw files outside the application repos unless they are small and intentionally versioned.

---

## 4. PDF extraction and audit pattern

For PDFs, extract page-level text and one combined text file. Store under:

```text
D:\JPISAI\data-processing\extracted-text\legal-acts\<country_or_scope>\<law_id>\
```

ZOO example output:

```json
{
  "source_pdf": "D:\\JPISAI\\data-raw\\legal-acts\\bih\\zoo_bih\\zoo_fbih_rs.pdf",
  "pages": 1265,
  "text_file": "D:\\JPISAI\\data-processing\\extracted-text\\legal-acts\\bih\\zoo_bih\\zoo_bih_extracted.txt",
  "pages_json": "D:\\JPISAI\\data-processing\\extracted-text\\legal-acts\\bih\\zoo_bih\\zoo_bih_pages.json"
}
```

Audit extraction before building JSON:

```text
- total_chars
- article marker count
- number of pages
- pages containing important articles
- OCR noise / bad character count
- count of notes/commentary/case-law markers
- whether important articles are found, e.g. 103 and 104
```

For ZOO, raw extraction first showed only 28 `Član` markers because the PDF contained heavy annotations and page/case-law noise. The solution was to normalize page text and extract heading-based article candidates.

---

## 5. Normalization and article candidate extraction

For each law create review scripts under:

```text
D:\JPISAI\data-processing\review\legal-acts\bih\<law_id>\
```

ZOO scripts used locally:

```text
extract_zoo_pdf_text.py

audit_zoo_extraction.py
diagnose_zoo_pages.py
normalize_zoo_text.py
extract_zoo_heading_candidates.py
audit_zoo_duplicates.py
build_zoo_fbih_preview_dataset.py
patch_zoo_fbih_titles.py
```

Core article extraction rule:

```text
Use heading-like markers such as:
Član 103.
Član 104.

Do not blindly split on every number. Annotated PDFs contain case law references, comments, examples, and footnotes that can look like articles.
```

For annotated sources, remove or stop before:

```text
Napomena
Sudska praksa
Iz obrazloženja
http://
www.
long separators
example/form templates such as "Primjer broj ..."
```

Also normalize soft hyphen line breaks:

```text
pra-\nvičan -> pravičan
```

---

## 6. Duplicate article resolution

Duplicate article numbers are expected in amended or FBiH/RS split texts.

ZOO FBiH example:

```text
total heading candidates: 1119
unique article numbers: 1109
duplicates: 1, 2, 10, 11, 14, 395, 455, 557, 1104, 1105
```

For ZOO FBiH operational preview the chosen duplicate resolution was:

```text
1    -> variant 2
2    -> variant 2
10   -> variant 2
11   -> variant 2
14   -> variant 2
395  -> variant 1
455  -> variant 1
557  -> variant 1
1104 -> variant 1
1105 -> variant 1
```

Always write a build report with:

```text
selected_articles
unique_article_numbers
missing_numeric_articles
empty_or_too_short_articles
duplicates_resolved
article_103 preview
article_104 preview
```

For ZOO FBiH, the required build acceptance was:

```text
selected_articles: 1109
unique_article_numbers: 1109
missing_numeric_articles_1_1109: []
empty_or_too_short_articles_count: 0
```

---

## 7. Controlled preview dataset structure

Use a controlled preview root before touching the backend:

```text
D:\JPISAI\data-processing\review\legal-acts\bih\zoo_bih\controlled-ingestion-preview-root\
  laws\
    zoo_fbih\
      law.json
      articles\
        clan_0001_1.json
        clan_0002_2.json
        ...
        clan_1109_1109.json
```

`law.json` should contain source metadata and status:

```json
{
  "id": "zoo_fbih",
  "title": "Zakon o obligacionim odnosima Federacije Bosne i Hercegovine",
  "short_title": "ZOO FBiH",
  "jurisdiction": "FBiH",
  "type": "law",
  "status": "operational_preview"
}
```

Each article JSON should contain:

```json
{
  "id": "zoo_fbih_clan_103",
  "law_id": "zoo_fbih",
  "article_number": "103",
  "title": "Ništavost",
  "content": "...",
  "paragraphs": ["..."],
  "keywords": [],
  "metadata": {
    "sort_key": 103
  }
}
```

---

## 8. UTF-8 no-BOM rewrite

Before running the TypeScript ingestion pipeline, rewrite JSON files without BOM:

```powershell
$root = "D:\JPISAI\data-processing\review\legal-acts\bih\zoo_bih\controlled-ingestion-preview-root"
$jsonFiles = Get-ChildItem $root -Recurse -Filter *.json -File
$utf8NoBom = New-Object System.Text.UTF8Encoding($false)

foreach ($f in $jsonFiles) {
  $content = [System.IO.File]::ReadAllText($f.FullName)
  [System.IO.File]::WriteAllText($f.FullName, $content, $utf8NoBom)
}

"Rewritten JSON files without BOM: $($jsonFiles.Count)"
```

---

## 9. Dry-run before backend push

From `jpis-ingestion`:

```powershell
Set-Location "D:\JPISAI\jpis-ingestion"

$env:JPIS_DATASETS_ROOT = "D:\JPISAI\data-processing\review\legal-acts\bih\zoo_bih\controlled-ingestion-preview-root"
$env:JPIS_BACKEND_URL = "https://ai-jpis-ba-backend.onrender.com"

npm run dev:law 2>&1 | Tee-Object -FilePath .\zoo_fbih-dry-run.log
```

Check the log tail:

```powershell
Get-Content .\zoo_fbih-dry-run.log -Tail 40

Select-String -Path .\zoo_fbih-dry-run.log -Pattern "Error|Invalid|failed|throw|Cannot|ENOENT|AJV"
```

For ZOO, the dry-run had to reach:

```text
zoo_fbih/articles/clan_1109_1109.json
```

Only push after the dry-run has no validation errors.

---

## 10. Live import into backend

Set env for the current PowerShell session only:

```powershell
Set-Location "D:\JPISAI\jpis-ingestion"

$env:JPIS_DATASETS_ROOT = "D:\JPISAI\data-processing\review\legal-acts\bih\zoo_bih\controlled-ingestion-preview-root"
$env:JPIS_BACKEND_URL = "https://ai-jpis-ba-backend.onrender.com"
$env:JPIS_IMPORT_API_KEY = "<paste key from Render env or local .env>"

npm run dev:law -- --push 2>&1 | Tee-Object -FilePath .\zoo_fbih-qdrant-push.log
```

Do not commit real keys to GitHub. If a key appears in logs or screenshots, rotate it before production.

---

## 11. Large law import warning

Large laws can time out if imported in one request with indexing enabled.

For ZOO FBiH, the first full push sent:

```text
lawsCount: 1
articlesCount: 1109
options: { index: true }
```

The request timed out while the backend was still processing DB upserts, chunk creation, embeddings, and Qdrant upsert. Result: partial import.

Common symptoms:

```text
TypeError: fetch failed
UND_ERR_HEADERS_TIMEOUT
```

This can happen even if the backend keeps working for a while. Always verify the DB count after a failed push.

---

## 12. Verify law and article counts through backend API

Use the law detail route to get the internal law id:

```powershell
$base = "https://ai-jpis-ba-backend.onrender.com"
$detail = Invoke-RestMethod "$base/jpis/laws/zoo_fbih"
$lawId = $detail.law.id

$countCheck = Invoke-RestMethod "$base/jpis/law-articles?law_id=$lawId&take=1"
"TOTAL ARTICLES IN DB: $($countCheck.count)"
```

Important: `/jpis/law-articles` filters by the internal `law.id`, not necessarily the canonical id.

Verify important articles:

```powershell
Invoke-RestMethod "$base/jpis/law-articles?law_id=$lawId&q=103&take=5" | ConvertTo-Json -Depth 10
Invoke-RestMethod "$base/jpis/law-articles?law_id=$lawId&q=104&take=5" | ConvertTo-Json -Depth 10
Invoke-RestMethod "$base/jpis/law-articles?law_id=$lawId&q=1108&take=5" | ConvertTo-Json -Depth 10
```

ZOO accepted result:

```text
TOTAL ARTICLES IN DB: 1109
article 103 found
article 104 found
article 1108 found
```

---

## 13. Repair partial imports

If the count is lower than expected, calculate missing articles:

```powershell
$base = "https://ai-jpis-ba-backend.onrender.com"
$detail = Invoke-RestMethod "$base/jpis/laws/zoo_fbih"
$lawId = $detail.law.id

$all = @()
for ($skip = 0; $skip -lt 2000; $skip += 200) {
  $page = Invoke-RestMethod "$base/jpis/law-articles?law_id=$lawId&take=200&skip=$skip"
  $all += $page.law_articles
  if ($page.law_articles.Count -lt 200) { break }
}

$existing = $all | ForEach-Object { [int]$_.article_number } | Sort-Object -Unique
$missing = 1..1109 | Where-Object { $existing -notcontains $_ }

"Existing articles: $($existing.Count)"
"Missing articles: $($missing.Count)"
"Missing now: $($missing -join ', ')"
```

For DB-only repair, send `options.index=false`.

If chunking itself causes failures or creates unnecessary load, send:

```json
{
  "options": {
    "index": false,
    "create_chunks": false
  }
}
```

This requires backend support for `options.create_chunks=false`.

---

## 14. Use curl.exe --data-binary for final repair payloads

PowerShell `Invoke-RestMethod -Body $body` can mangle or re-encode JSON text in a way that causes backend `body-parser` errors such as:

```text
Expected ',' or '}' after property value in JSON
```

When this happens, write the payload to a UTF-8 no-BOM file, validate locally, and send with `curl.exe --data-binary`:

```powershell
$json = $payload | ConvertTo-Json -Depth 100
$utf8NoBom = New-Object System.Text.UTF8Encoding($false)
[System.IO.File]::WriteAllText($payloadPath, $json, $utf8NoBom)

# local validation
$null = [System.IO.File]::ReadAllText($payloadPath, [System.Text.Encoding]::UTF8) | ConvertFrom-Json

& curl.exe `
  -sS `
  -X POST "$base/jpis/import/laws" `
  -H "content-type: application/json; charset=utf-8" `
  -H "x-jpis-import-key: $env:JPIS_IMPORT_API_KEY" `
  --data-binary "@$payloadPath" `
  -o "$responsePath" `
  -w "%{http_code}"
```

For ZOO, the final 10 articles were imported successfully this way with `create_chunks=false`.

---

## 15. Backend details that matter

The backend law import endpoint:

```text
POST /jpis/import/laws
```

Main behavior:

```text
- upsert source
- upsert law
- upsert entity document
- upsert law articles
- optionally create document chunks
- optionally create embeddings and Qdrant points
```

Important options:

```json
{
  "index": true,
  "create_chunks": true
}
```

Defaults:

```text
index: true
create_chunks: true
```

Repair mode:

```json
{
  "index": false,
  "create_chunks": false
}
```

This inserts/updates law and article records only.

---

## 16. Frontend/detail route caveat

The law may be complete in DB while the frontend shows only 500 articles. This is caused by the backend detail route if it fetches articles with:

```ts
pagination: { take: 500 }
```

Required backend fix:

```text
GET /jpis/laws/[id] must paginate all articles for large laws and return 1109 for ZOO FBiH.
```

Verification after backend fix:

```powershell
$base = "https://ai-jpis-ba-backend.onrender.com"
$detail = Invoke-RestMethod "$base/jpis/laws/zoo_fbih"
"DETAIL ARTICLES RETURNED: $($detail.articles.Count)"
```

Expected:

```text
DETAIL ARTICLES RETURNED: 1109
```

---

## 17. What not to do

Do not:

- push a large law with `index=true` blindly in one request
- ignore a timeout and assume import failed completely
- trust frontend count without checking `/jpis/law-articles` count
- paste PowerShell prompts (`PS D:\...>`) into commands
- commit import keys to GitHub docs or scripts
- commit raw temporary repair payloads containing secrets
- re-import a completed law unless there is a deliberate correction plan

---

## 18. Minimum acceptance checklist for a law

A law import is complete only when all are true:

```text
[ ] source file downloaded and recorded
[ ] extraction audit generated
[ ] article candidate count reviewed
[ ] duplicate articles resolved
[ ] controlled preview dataset built
[ ] JSON rewritten UTF-8 no-BOM
[ ] dry-run completed without schema errors
[ ] backend DB count equals expected article count
[ ] important articles verified by API
[ ] frontend/detail route returns all articles or has a tracked backend fix
[ ] import key not committed and scheduled for rotation before production
```
