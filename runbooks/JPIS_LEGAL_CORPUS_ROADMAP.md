# JPIS roadmap pravnog korpusa

## Svrha

JPIS gradi kompletan pravni korpus, a ne samo skup zakona koji su citirani u početnom golden setu presuda.

Presude služe za prioritizaciju, povezivanje članova, validaciju parsera i provjeru kvaliteta nakon ingestije. One ne odlučuju da li pravni akt ulazi u korpus. Dugoročni korpus treba obuhvatiti ključne pravne akte Bosne i Hercegovine, Federacije Bosne i Hercegovine, Republike Srpske, Brčko distrikta Bosne i Hercegovine, kao i relevantne međunarodne i EU izvore.

Ovaj dokument je operativni roadmap i backlog okvir. Nije konačan popis svih pravnih akata i nije službeni pravni katalog.

## Jurisdikcije i izvori

- Bosna i Hercegovina
- Federacija Bosne i Hercegovine
- Republika Srpska
- Brčko distrikt Bosne i Hercegovine
- Međunarodni izvori relevantni za Bosnu i Hercegovinu
- EU pravo relevantno za Bosnu i Hercegovinu i usklađivanje sa acquis-em

## Model statusa

Status pravnog akta u JPIS-u ne smije miješati pravnu važnost akta, fazu obrade korpusa i tehničku oznaku kvaliteta dataseta. Za buduće akte koriste se tri odvojena polja.

### `legal_status` - pravni status akta

`legal_status` odgovara na pitanje:

> Da li akt trenutno pravno važi i kakav je njegov normativni položaj?

Vrijednosti:

- `active` - akt je trenutno na snazi
- `repealed` - akt je ukinut
- `superseded` - akt je zamijenjen drugim aktom ili novijom regulacijom
- `historical` - akt se čuva zbog historijskog, prijelaznog ili sudsko-praktičnog značaja
- `unknown` - pravni status nije potvrđen

### `corpus_status` - status obrade akta u JPIS-u

`corpus_status` odgovara na pitanje:

> U kojoj se fazi pribavljanja, obrade, validacije ili dostupnosti akt nalazi unutar JPIS-a?

Vrijednosti:

- `planned` - akt je identificiran kao dio backlog-a korpusa
- `source_discovery` - traže se kandidatski izvori
- `source_verified` - potvrđen je autoritativni ili operativno prihvatljiv izvor
- `extraction` - ekstrakcija teksta je u toku
- `processing` - segmentacija članova, čišćenje, metadata i pregled su u toku
- `dataset_ready` - canonical dataset fajlovi su pripremljeni i lokalno validirani
- `imported` - akt je unesen u JPIS backend
- `operational_preview` - akt je dostupan kao operativni preview, uz jasno označena ograničenja
- `validated` - akt je prošao post-ingestion provjere, uključujući broj članova, API/DB provjeru i osnovnu frontend provjeru
- `needs_review` - potreban je dodatni pravni, metadata, izvorni, ekstrakcijski ili article-level pregled

### `review_status` - oznaka kvaliteta i nivoa pregleda

`review_status` je slobodna, ali dokumentovana tehnička oznaka koja opisuje kvalitet, verziju obrade i nivo izvršenog pregleda dataseta.

`review_status` odgovara na pitanje:

> Koji je nivo pregleda, normalizacije, segmentacije i provjere izvršen nad konkretnim datasetom?

Primjeri:

- `baseline_segmented_clean_v4`
- `consolidated_latest_preview_ready`

Napomene:

- Postojeći dataset model koristi `status: active` kao pravni status.
- `metadata.review_status` nosi oznaku kvaliteta i nivoa pregleda.
- Tehničke vrijednosti ostaju na engleskom jer su dio stabilnog modela podataka, ali njihova objašnjenja u dokumentaciji moraju biti na bosanskom jeziku.

Primjer za ZKP BiH:

```text
Pravni status: `active`
Status korpusa: `imported`
Oznaka pregleda: `baseline_segmented_clean_v4`
```

ZKP BiH pravno važi, već je unesen u JPIS i ima bazni operativni nivo obrade i pregleda.

## Ingestion valovi

### Val 1: temeljni propisi

- Krivično materijalno i procesno pravo
- Građansko pravo i obligacije
- Parnični postupak
- Upravni postupak
- Upravni spor
- Izvršni postupak
- Stvarna prava i vlasništvo
- Privredna društva
- Radni odnosi

### Val 2: pravosuđe i pravna profesija

- Sudovi
- Tužilaštva
- Visoko sudsko i tužilačko vijeće
- Advokatura i pravna profesija
- Notari
- Sudske takse
- Izvršitelji
- Vještaci
- Besplatna pravna pomoć

### Val 3: sektorsko pravo

- Prostorno uređenje
- Građenje
- Porezi
- Javne nabavke
- Zaštita potrošača
- Zaštita ličnih podataka
- Porodično pravo
- Nasljeđivanje
- Javna uprava
- Privreda i finansije

### Val 4: međunarodno i EU pravo

- Evropska konvencija o ljudskim pravima i protokoli
- Ključne UN konvencije
- Sporazum o stabilizaciji i pridruživanju
- Relevantne EU uredbe i direktive
- Pravosudna saradnja
- Digitalni dokazi
- `GDPR` i zaštita podataka
- `AI` i kibernetička sigurnost

## Trenutno završeni akti u JPIS-u

| Canonical ID | Kratki naziv | Poznati status |
| --- | --- | --- |
| `criminal_code_bih` | KZ BiH | `consolidated_latest_preview_ready` |
| `criminal_procedure_code_bih` | ZKP BiH | `baseline_segmented_clean_v4` |
| `zoo_fbih` | ZOO FBiH | `operational_preview` |
| `zpp_fbih` | ZPP FBiH | `operational_preview` |
| `zpd_fbih` | ZPD FBiH | `operational_preview` |

Ovi akti se smatraju završenim za trenutno odobreni operativni nivo. Oznake statusa opisuju nivo provjere i obrade, a ne znače da akt nije upotrebljiv u trenutnom sistemu.

## Sljedeći kandidati iz Vala 1

- Zakon o parničnom postupku Republike Srpske
- Zakon o parničnom postupku Brčko distrikta BiH
- Zakon o upravnim sporovima Republike Srpske
- Zakon o opštem upravnom postupku Republike Srpske
- Zakon o stvarnim pravima Republike Srpske
- Relevantni Brčko upravni i procesni propisi

Ovo nisu jedini zakoni koji će se unositi. To su sljedeći operativni kandidati za obradu na osnovu potrebe za pokrivenošću korpusa i validacijskih signala iz presuda.

## Standard obrade svakog akta

Svaki pravni akt treba proći kroz kontrolisani workflow:

1. Identificirati autoritativni izvor.
2. Evidentirati službeni glasnik i historiju izmjena.
3. Provesti extraction audit.
4. Izgraditi kontrolisani preview dataset.
5. Provjeriti `UTF-8` bez `BOM` oznake.
6. Pokrenuti dry-run ingestije.
7. Tražiti izričito odobrenje prije production importa.
8. Provjeriti API i DB brojčano stanje nakon production importa.
9. Provjeriti frontend list/detail prikaz.
10. Pokrenuti post-ingestion validaciju povezivanja članova preko presuda.

## Pravila sigurnosti

- Nikad ne pokretati `git reset --hard`.
- Nikad ne pokretati `git clean`.
- Nikad ne koristiti `git add .`.
- Nikad ne raditi import bez čistog dry-run rezultata.
- Nikad ne pisati u production API ili DB bez izričitog odobrenja.
- Nikad ne ispisivati `JPIS_IMPORT_API_KEY` ili bilo koju ekvivalentnu tajnu vrijednost.
- Nikad ne brisati lokalne `jpis-ingestion` logove ili outpute bez posebnog review-a.
