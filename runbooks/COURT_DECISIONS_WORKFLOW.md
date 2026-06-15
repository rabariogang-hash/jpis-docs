# COURT DECISIONS WORKFLOW

## Svrha

Ovaj workflow definiše kako se u JPIS AI uvode sudske odluke kao strukturirani analitički objekti, a ne samo kao sirovi tekstovi.

Presude služe za:
- analizu sudske prakse
- povezivanje sudskih odluka sa zakonima i člancima
- kasniji AI sloj za pravnu analizu
- pretragu po sudu, broju predmeta, sažetku, činjenicama i obrazloženju
- kasniji linking sa KZ BiH, ZKP BiH, drugim zakonima i relevantnim konvencijama

## Glavni princip

Ne unositi presude masovno bez standarda.

Prvo mora postojati:
1. zaključan schema standard
2. golden example presude
3. pilot import sa 1 stvarnom presudom
4. provjera API + frontend + search ponašanja
5. tek onda širenje korpusa

## Folder logika

### Raw
D:\JPISAI\data-raw\court-decisions\{jurisdiction}\{court_slug}\{decision_slug}\

### Processing / review
D:\JPISAI\data-processing\review\court-decisions\{jurisdiction}\{court_slug}\{decision_slug}\

### Dataset
D:\JPISAI\jpis-datasets\court-decisions\{jurisdiction}\{court_slug}\{decision_slug}.json

## Minimalni obavezni elementi za stvarnu presudu

- id
- court_name
- jurisdiction
- case_number
- decision_date
- source

## Preporučeni operativni elementi

- summary
- full_text ili full_text_path
- anonymized_text ili anonymized_text_path
- structure
- referenced_articles
- referenced_laws
- keywords
- outcome
- citations
- metadata

## Golden example pravilo

Golden example se koristi kao tehnički šablon.
Ne push-a se na live backend dok ne bude zamijenjen stvarnim sadržajem presude.

## Pilot faza

Pilot faza podrazumijeva:
- 1 stvarnu presudu za prvi prolaz
- zatim ukupno 3 ručno sređene pilot presude
- provjeru da frontend prikazuje:
  - listu presuda
  - detail presude
  - strukturirane blokove
  - citate i metapodatke

## Law linking pravilo

Kada god je moguće, presuda treba imati:
- referenced_laws
- referenced_articles

Primjeri:
- criminal_code_bih
- criminal_procedure_code_bih

## Napomena o authoritative statusu

Presude i njihovi strukturirani prikazi ne smiju se predstavljati kao službeno uređeni ili autoritativno prečišćeni tekst ako za to nema osnova.

## Operativni cilj ove faze

Cilj ove faze nije masovni import, nego:
- zaključati standard
- potvrditi pipeline
- potvrditi frontend/API ponašanje
- pripremiti sistem za ozbiljan korpus sudske prakse