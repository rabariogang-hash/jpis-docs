# JPIS Retrieval Guardrails

Datum: 2026-06-23

Ovaj runbook opisuje P0 zaštite koje moraju biti aktivne prije masovne ingestije zakona u JPIS. Cilj je spriječiti miješanje članova istog broja iz različitih zakona, jurisdikcija, pravnih oblasti ili verzija propisa.

## P0 Status

P0 zaštite se smatraju obaveznim uslovom za batch pipeline:

- backend mora podržavati deterministički dohvat člana po `law_canonical_id + article_number`;
- AI/Qdrant retrieval mora prihvatiti opcione filtere `law_canonical_id`, `jurisdiction`, `article_number` i `legal_domain`;
- runtime parser mora imati centralni registry aktivnih zakona i ne smije tiho fallbackati na FBiH kada je naziv zakona dvosmislen;
- dataset repo mora imati registry aktivnih pravnih akata sa aliasima, jurisdikcijom i pravnom oblasti;
- svaki novi zakon mora imati `aliases` i `metadata.legal_domain` prije canonical commita.

Batch pipeline ne smije preći u production import ako P0 testovi ili lokalni dry-run pokažu da se član može vezati za pogrešan zakon.

## Aktivni Zakoni U Registryju

Trenutni P0 registry obuhvata:

- `criminal_code_bih` - krivično materijalno pravo, BiH;
- `criminal_procedure_code_bih` - krivični postupak, BiH;
- `zoo_fbih` - obligaciono pravo, FBiH;
- `zpp_fbih` - parnični postupak, FBiH;
- `zpd_fbih` - privredna društva, FBiH;
- `zpp_bdbih` - parnični postupak, Brčko distrikt BiH;
- `zip_fbih` - izvršni postupak, FBiH.

Za `zoo_fbih`, `zpp_fbih` i `zpd_fbih` produkcijski podaci postoje, ali lokalni canonical snapshot još treba parity audit prije oslanjanja na repo kao jedini izvor istine.

## Backend Pravila

Backend treba omogućiti read-only lookup člana preko javnog canonical ID-ja zakona:

```text
law_canonical_id + article_number
```

Primjeri:

```text
zip_fbih + 79a
zpp_bdbih + 457h
criminal_procedure_code_bih + 84
```

Lookup mora prvo pronaći zakon po canonical ID-ju, a zatim član isključivo unutar tog zakona. Ako zakon ili član ne postoje, rezultat mora biti `404`. Sistem ne smije vraćati član istog broja iz drugog zakona.

## Retrieval Filteri

AI/Qdrant retrieval mora podržati opcione filtere:

```text
law_canonical_id
jurisdiction
article_number
legal_domain
```

Bez filtera postojeće ponašanje ostaje kompatibilno. Sa `law_canonical_id`, rezultat mora biti ograničen na taj zakon. Sa `jurisdiction + article_number`, sistem ne smije vratiti član iz druge jurisdikcije.

## Runtime Pravila

Runtime registry mora sadržavati `canonical_id`, službeni naziv, skraćeni naziv, alias-e, jurisdikciju, pravnu oblast i oznaku linkabilnosti.

Posebno pravilo za dvosmislene nazive:

- `ZPP BD BiH` i Brčko kontekst ne smiju linkati `zpp_fbih`;
- `ZIP FBiH` i član `79a` moraju linkati `zip_fbih`;
- eksplicitni `KZ BiH`, `ZKP BiH`, `ZOO FBiH`, `ZPP FBiH`, `ZPD FBiH`, `ZPP BD BiH` i `ZIP FBiH` moraju biti prepoznatljivi;
- goli `ZPP` bez jurisdikcije ne smije tiho fallbackati na FBiH kada postoji više kandidata.

## Dataset Pravila

Svaki novi `law.json` treba imati:

```text
aliases
metadata.legal_domain
metadata.legal_status
metadata.corpus_status
metadata.review_status
metadata.version_scope
metadata.source_confidence
```

Tehničke vrijednosti ostaju stabilni identifikatori na engleskom jeziku. Objašnjenja, napomene i runbook tekst moraju biti na bosanskom jeziku, latinica.

Centralni registry se nalazi u:

```text
jpis-datasets/registry/legal-acts-registry.json
```

Registry je guardrail sloj i ne zamjenjuje canonical `law.json` fajlove.

## P1 Reindex I Backfill Plan

P0 uvodi filtere i metadata standard, ali postojeći Qdrant pointovi mogu imati stariji payload bez svih novih polja. P1 mora uraditi kontrolisani backfill/reindex:

1. Napraviti read-only audit postojećih Qdrant payload polja za zakone i članke.
2. Utvrditi koji pointovi već imaju `law.canonical_id`, `law.jurisdiction`, `article.article_number` i `law.legal_domain`.
3. Pripremiti idempotentni reindex plan koji dopunjava nedostajuće payload vrijednosti iz canonical dataset registryja.
4. Pokrenuti staging ili lokalni dry-run prije production reindexa.
5. Production reindex raditi samo uz izričito odobrenje i sa rollback evidencijom.

Dok P1 nije završen, retrieval filteri moraju biti tretirani kao dostupni za nove/importovane podatke i kao djelimično zavisni od starog payload kvaliteta za ranije indeksirane podatke.

## Regression Testovi

Minimalni testovi prije batch ingestije:

- isti broj člana u dva različita zakona mora se dohvatiti samo kroz traženi `law_canonical_id`;
- Brčko referenca `član 240. ZPP` u Brčko kontekstu ne smije linkati `zpp_fbih`;
- `član 79a. ZIP FBiH` mora linkati `zip_fbih`, član `79a`;
- eksplicitna referenca poput `KZ BiH član 42` mora koristiti canonical zakon prije semantic searcha;
- upit bez jurisdikcije kod dvosmislenog naziva ne smije automatski izabrati FBiH;
- svaki registry zapis mora imati canonical ID, jurisdikciju, pravnu oblast i barem jedan alias.

## Sigurnosna Pravila

- Bez `git reset --hard`.
- Bez `git clean`.
- Bez `git add .`.
- Bez production import-a bez prethodnog dry-run-a.
- Bez production API/DB write-a bez izričitog odobrenja.
- Bez ispisivanja `JPIS_IMPORT_API_KEY`.
- Bez brisanja lokalnih ingestion logova i outputa bez posebnog review-a.
