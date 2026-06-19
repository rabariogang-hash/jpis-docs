# JPIS Legal Corpus Roadmap

## Purpose

JPIS is building a complete legal corpus, not only a set of laws cited in the initial golden court-decision set.

Court decisions are used for prioritization, article linking, parser validation, and post-ingestion quality checks. They do not decide whether a legal act belongs in the corpus. The long-term corpus should cover key legal acts from Bosnia and Herzegovina, the Federation of Bosnia and Herzegovina, Republika Srpska, Brcko District of Bosnia and Herzegovina, and relevant international and EU sources.

This document is an operational roadmap and backlog framework. It is not a final list of all legal acts and it is not an official legal catalogue.

## Jurisdictions and Source Domains

- Bosnia and Herzegovina
- Federation of Bosnia and Herzegovina
- Republika Srpska
- Brcko District of Bosnia and Herzegovina
- International sources relevant to Bosnia and Herzegovina
- EU law relevant to Bosnia and Herzegovina and acquis alignment

## Status Model

Each future legal act should carry one explicit operational status:

- `planned` - identified as part of the corpus backlog
- `source_discovery` - candidate sources are being located
- `source_verified` - authoritative or operationally acceptable source is confirmed
- `extraction` - text extraction is in progress
- `processing` - article segmentation, cleanup, metadata, and review are in progress
- `dataset_ready` - canonical dataset files are prepared and validated locally
- `operational_preview` - imported or usable as an operational preview, with explicit limitations
- `active` - active production corpus item at its approved verification level
- `consolidated_latest_preview_ready` - consolidated latest preview is available, with non-authoritative status clearly marked
- `archived_or_repealed` - archived, repealed, or superseded act retained for historical/legal context
- `needs_review` - requires legal, metadata, source, extraction, or article-level review

## Ingestion Waves

### Wave 1: Foundational Legal Acts

- Criminal substantive and procedural law
- Civil law and obligations
- Civil procedure
- Administrative procedure
- Administrative disputes
- Enforcement procedure
- Real rights and ownership
- Companies and business entities
- Labor relations

### Wave 2: Judiciary and Legal Profession

- Courts
- Prosecutor's offices
- High Judicial and Prosecutorial Council
- Advocacy/legal profession
- Notaries
- Court fees
- Enforcement officers
- Expert witnesses
- Free legal aid

### Wave 3: Sectoral Law

- Spatial planning
- Construction
- Taxes
- Public procurement
- Consumer protection
- Personal data protection
- Family law
- Inheritance
- Public administration
- Economy and finance

### Wave 4: International and EU Law

- European Convention on Human Rights and protocols
- Key UN conventions
- Stabilisation and Association Agreement
- Relevant EU regulations and directives
- Judicial cooperation
- Digital evidence
- GDPR and data protection
- AI and cybersecurity

## Currently Completed Acts in JPIS

| Canonical ID | Short title | Known status |
| --- | --- | --- |
| `criminal_code_bih` | KZ BiH | `consolidated_latest_preview_ready` |
| `criminal_procedure_code_bih` | ZKP BiH | `baseline_segmented_clean_v4` |
| `zoo_fbih` | ZOO FBiH | `operational_preview` |
| `zpp_fbih` | ZPP FBiH | `operational_preview` |
| `zpd_fbih` | ZPD FBiH | `operational_preview` |

These acts are considered complete for their currently approved operational status. The status labels describe verification and processing level, not whether the act is useful in the current system.

## Next Wave 1 Batch Candidates

- Zakon o parnicnom postupku Republike Srpske
- Zakon o parnicnom postupku Brcko distrikta BiH
- Zakon o upravnim sporovima Republike Srpske
- Zakon o opstem upravnom postupku Republike Srpske
- Zakon o stvarnim pravima Republike Srpske
- Relevant Brcko administrative and procedural regulations

These are not the only laws that will be ingested. They are the next operational candidates for processing based on corpus coverage needs and court-decision validation signals.

## Standard Processing Workflow for Each Act

Each legal act should pass through the following controlled workflow:

1. Identify an authoritative source.
2. Record official gazette references and amendment history.
3. Run extraction audit.
4. Build a controlled preview dataset.
5. Verify UTF-8 without BOM.
6. Run dry-run ingestion.
7. Require explicit approval before production import.
8. Verify API and DB counts after production import.
9. Verify frontend list/detail behavior.
10. Run post-ingestion article-link validation against court decisions.

## Safety Rules

- Never run `git reset --hard`.
- Never run `git clean`.
- Never run `git add .`.
- Never import without a clean dry-run first.
- Never write to production API or DB without explicit approval.
- Never print `JPIS_IMPORT_API_KEY` or any equivalent secret value.
- Never delete local `jpis-ingestion` logs or outputs without a dedicated review.
