# JPIS plan korpusa, retrievala i masovne obrade

## 1. Svrha

JPIS se gradi kao jedan federativni pravni informacioni sistem za Bosnu i Hercegovinu, Federaciju Bosne i Hercegovine, Republiku Srpsku, Brčko distrikt Bosne i Hercegovine, relevantno međunarodno pravo i odabrano pravo Evropske unije.

Cilj nije napraviti nepovezane AI sisteme po entitetima ili granama prava. Cilj je jedan sistem sa strogo kontrolisanim pravnim routingom, metadata filtrima i dokazivim izvorima. Time se omogućava rad na predmetima koji istovremeno uključuju više jurisdikcija i oblasti, bez miješanja članova iz različitih propisa.

Ovaj dokument definiše:

- model koji smanjuje rizik pogrešnog miješanja propisa;
- redoslijed proširenja korpusa;
- minimalni pravni metadata standard;
- pravila za batch obradu zakona;
- operativni katalog ključnih akata za JPIS.

Ovo je operativni backlog i ne predstavlja zatvoren ili konačan katalog svakog propisa u BiH. Tačan službeni naziv, pravni status, niz izmjena i primjenjivost svakog konkretnog akta potvrđuju se prije njegove ingestije.

## 2. Ključna arhitektonska odluka

JPIS ostaje jedan sistem, ali retrieval se vodi kroz obavezne filtere.

Ne graditi zaseban AI za FBiH, RS, Brčko ili svaku granu prava. Takva podjela bi otežala:

- predmete sa više pravnih oblasti;
- poređenje propisa između jurisdikcija;
- održavanje presuda i referenci;
- upravljanje kvalitetom i verzijama;
- jedinstveno korisničko iskustvo.

Umjesto toga, svaki zakon, član, paragraf i indeksirani chunk mora naslijediti identitet nadređenog propisa.

## 3. Obavezni metadata model

### 3.1. Na nivou zakona

Svaki `law.json` treba podržati najmanje:

```json
{
  "id": "criminal_code_fbih",
  "title": "Krivični zakon Federacije Bosne i Hercegovine",
  "short_title": "KZ FBiH",
  "aliases": [
    "KZ FBiH",
    "Krivični zakon FBiH",
    "Krivični zakon Federacije BiH"
  ],
  "jurisdiction": "fbih",
  "type": "law",
  "status": "operational_preview",
  "metadata": {
    "legal_domain": "criminal_substantive",
    "legal_status": "active",
    "corpus_status": "imported",
    "review_status": "...",
    "version_scope": "...",
    "official_publication_references": []
  }
}
```

### 3.2. Na nivou člana i indeksiranog chunka

Svaki član i svaki chunk koji ide u search/index mora nositi najmanje:

```text
law_id
law_title
law_aliases
jurisdiction
legal_domain
legal_status
corpus_status
review_status
article_number
article_title
version_scope
effective_from
effective_to
source_confidence
```

`law_id`, `jurisdiction`, `legal_domain` i `article_number` ne smiju se izgubiti tokom chunkinga, embeddinga, import procesa ili API transformacije.

### 3.3. Na nivou presude

Za presude se čuvaju i koriste:

```text
court_jurisdiction
court_level
procedure_type
applied_law_ids
cited_law_references
cited_article_references
decision_date
```

Kod eksplicitne reference, npr. `član 42. KZ FBiH`, sistem prvo mora pokušati determinističko povezivanje:

```text
criminal_code_fbih + 42
```

Tek nakon toga dopušten je semantički retrieval za dodatni kontekst.

## 4. Pravila retrievala i analize presuda

### 4.1. Retrieval prije semantičke pretrage

JPIS prvo filtrira po:

1. jurisdikciji;
2. grani prava / `legal_domain`;
3. vremenski relevantnoj verziji propisa;
4. pravnom statusu;
5. eksplicitno citiranim zakonima i članovima;
6. tek zatim radi semantičku pretragu unutar suženog skupa.

Nije dopušteno da generički semantic search bira između svih članova sa istim brojem iz više jurisdikcija kada je jurisdikcija poznata ili se može utvrditi iz presude.

### 4.2. Redoslijed analize presude

```text
presuda
→ identifikacija suda, jurisdikcije i postupka
→ ekstrakcija eksplicitno citiranih propisa i članova
→ determinističko povezivanje sa canonical law_id + article_number
→ provjera verzije relevantne na datum odluke / događaja
→ retrieval samo iz potvrđenih i povezanih propisa
→ po potrebi kontrolisano proširenje na povezane propise
→ obrazložena analiza sa izvorima
```

### 4.3. Quality gates prije širokog korištenja novog korpusa

Novi zakon ne ulazi automatski u retrieval za kompleksnu analitiku samo zato što je importovan. Potrebni su:

- schema validacija;
- brojčana provjera članova;
- `content_path` provjera;
- testovi canonical reference resolutiona;
- testovi protiv golden seta presuda;
- uzorak upita koji namjerno razlikuju FBiH, RS, Brčko i BiH nivo.

## 5. Standard izvora i transparentnost

### 5.1. Operativni model izvora

Za rast korpusa primjenjuje se sljedeće pravilo:

```text
javni digitalni tekst
+ službeni indeks objava / izmjena, kada je dostupan
→ operational_preview
→ kasniji upgrade na viši nivo izvora ili verifikacije
```

Javno dostupni HTML, PDF i DOCX izvori mogu se koristiti za operativni corpus ako se transparentno evidentiraju porijeklo, format, datum preuzimanja, hash, poznati niz objava i ograničenja.

Nije dopušteno:

- zaobilaženje login-a, paywalla ili tehničke zaštite;
- automatsko preuzimanje cijelih pravnih baza ili portala;
- prikazivanje sekundarnog teksta kao službenog prečišćenog teksta;
- brisanje ili prepisivanje izvornog teksta bez tragova promjene.

### 5.2. Izvorne razine

- `official`: službeni tekst ili službeni dokument institucije;
- `institutional`: javni tekst parlamenta, ministarstva, suda ili druge javne institucije;
- `secondary`: javno dostupni stručni/digitalni radni izvor;
- `manual_authorized_acquisition_candidate`: tekst koji korisnik zakonito pribavlja ručno, bez automatizovanog preuzimanja.

## 6. Batch pipeline: šta se automatizuje, a šta ostaje pod kontrolom

### 6.1. Automatizovati

- registry zakona i izvora;
- javno preuzimanje pojedinačnih otvorenih dokumenata;
- hashiranje i source manifest;
- ekstrakciju HTML/PDF/DOCX;
- osnovnu segmentaciju po članu;
- generation preview dataseta;
- schema, numeričke i `content_path` provjere;
- razvrstavanje na `READY_FOR_TRANSFER`, `NEEDS_LIGHT_REVIEW` i `BLOCKED_OR_INCOMPLETE`;
- batch izvještaj sa razlogom neuspjeha po zakonu.

### 6.2. Ne automatizovati bez posebne odluke

- production import;
- pravno odlučivanje da je tekst službeno prečišćen;
- složenu konsolidaciju velikog broja izmjena bez jasnog izvornog lanca;
- popravke normativnog sadržaja;
- prepisivanje ili normalizaciju teksta koja može promijeniti smisao norme;
- OCR kvalitetnu potvrdu bez uzorkovanja i pregleda.

### 6.3. Operativni batch režim

1. Registry grupiše zakone prema formatu i izvoru.
2. Batch preuzima i priprema samo javno otvorene pojedinačne izvore.
3. Svaki zakon dobija odvojeni manifest, raw artefakt, preview i validation report.
4. Problem jednog zakona ne prekida cijeli batch.
5. `READY_FOR_TRANSFER` zakoni idu u izolovan canonical review.
6. Import se radi u malim eksplicitno odobrenim serijama, npr. 3–5 zakona, sa post-import API provjerom svakog zakona.

## 7. Faze implementacije

### Faza A — zaštite retrievala prije masovne ingestije

Prioritet prije masovne obrade:

- obavezno nasljeđivanje `law_id`, `jurisdiction`, `legal_domain` i `article_number` u svim indeksnim chunkovima;
- aliasi za zakone;
- deterministic resolver za eksplicitne reference zakona i članova;
- filtrirani retrieval prije semantic searcha;
- test set za kolizije istih članova u više jurisdikcija;
- prikaz izvora, statusa i verzije u odgovoru sistema.

### Faza B — batch pipeline MVP

- centralni registry;
- acquisition za javne HTML/PDF/DOCX izvore;
- postojeći preview format;
- osnovna segmentacija;
- validacija;
- batch izvještaj;
- bez automatskog production importa.

### Faza C — napredna obrada

- OCR queue za skenirane propise;
- kontrolisana konsolidacija izmjena;
- temporalni model verzija;
- diff između verzija zakona;
- pravno pregledani upgrade iz `operational_preview` u viši nivo verifikacije.

## 8. Trenutno uneseni zakoni

| Canonical ID | Kratki naziv | Jurisdikcija | Operativni status |
| --- | --- | --- | --- |
| `criminal_code_bih` | KZ BiH | `bih` | unesen |
| `criminal_procedure_code_bih` | ZKP BiH | `bih` | unesen |
| `zoo_fbih` | ZOO FBiH | `fbih` | unesen |
| `zpp_fbih` | ZPP FBiH | `fbih` | unesen |
| `zpd_fbih` | ZPD FBiH | `fbih` | unesen |
| `zpp_bdbih` | ZPP BD BiH | `bdbih` | unesen |
| `zip_fbih` | ZIP FBiH | `fbih` | unesen |

## 9. Operativni katalog zakona za JPIS

Ovaj katalog predstavlja prioritetni korpus. Svaki red je kandidat za registry; prije obrade se potvrđuju tačan naziv, jurisdikcija, status, verzija i izvori.

### 9.1. BiH nivo — temeljni i pravosudni propisi

- Krivični zakon Bosne i Hercegovine
- Zakon o krivičnom postupku Bosne i Hercegovine
- Zakon o Sudu Bosne i Hercegovine
- Zakon o Tužilaštvu Bosne i Hercegovine
- Zakon o Visokom sudskom i tužilačkom vijeću Bosne i Hercegovine
- Zakon o zaštiti svjedoka pod prijetnjom i ugroženih svjedoka
- Zakon o programu zaštite svjedoka u Bosni i Hercegovini
- Zakon o međunarodnoj pravnoj pomoći u krivičnim stvarima
- Zakon o izvršenju krivičnih sankcija, pritvora i drugih mjera Bosne i Hercegovine
- Zakon o upravnom postupku Bosne i Hercegovine
- Zakon o upravnim sporovima Bosne i Hercegovine
- Zakon o upravi Bosne i Hercegovine
- Zakon o državnoj službi u institucijama Bosne i Hercegovine
- Zakon o slobodi pristupa informacijama na nivou institucija Bosne i Hercegovine
- Zakon o zaštiti ličnih podataka
- Zakon o zaštiti od diskriminacije
- Zakon o ravnopravnosti spolova u Bosni i Hercegovini
- Zakon o javnim nabavkama Bosne i Hercegovine
- Zakon o zaštiti potrošača u Bosni i Hercegovini
- Zakon o sprečavanju pranja novca i finansiranja terorističkih aktivnosti
- Zakon o porezu na dodanu vrijednost
- Zakon o carinskoj politici Bosne i Hercegovine
- Zakon o carinskim prekršajima Bosne i Hercegovine
- Zakon o politici direktnih stranih ulaganja u Bosni i Hercegovini
- Zakon o Centralnoj banci Bosne i Hercegovine
- Zakon o komunikacijama Bosne i Hercegovine
- Zakon o elektronskom potpisu
- Zakon o elektronskom dokumentu
- Zakon o elektronskom pravnom i poslovnom prometu
- Zakon o autorskom i srodnim pravima
- Zakon o industrijskom vlasništvu
- Zakon o zaštiti konkurencije

### 9.2. Federacija Bosne i Hercegovine

#### Krivično pravo, pravosuđe i postupci

- Krivični zakon Federacije Bosne i Hercegovine
- Zakon o krivičnom postupku Federacije Bosne i Hercegovine
- Zakon o sudovima u Federaciji Bosne i Hercegovine
- Zakon o tužilaštvima Federacije Bosne i Hercegovine
- Zakon o zaštiti svjedoka u krivičnom postupku Federacije Bosne i Hercegovine
- Zakon o izvršenju krivičnih sankcija u Federaciji Bosne i Hercegovine
- Zakon o prekršajima Federacije Bosne i Hercegovine
- Zakon o pomilovanju Federacije Bosne i Hercegovine

#### Građansko, porodično i imovinsko pravo

- Zakon o obligacionim odnosima koji se primjenjuje u Federaciji Bosne i Hercegovine, uz relevantne izmjene i prijelazne izvore
- Zakon o parničnom postupku Federacije Bosne i Hercegovine
- Zakon o izvršnom postupku Federacije Bosne i Hercegovine
- Zakon o stvarnim pravima Federacije Bosne i Hercegovine
- Zakon o zemljišnim knjigama Federacije Bosne i Hercegovine
- Zakon o nasljeđivanju Federacije Bosne i Hercegovine
- Porodični zakon Federacije Bosne i Hercegovine
- Zakon o vanparničnom postupku Federacije Bosne i Hercegovine
- Zakon o zaštiti od nasilja u porodici Federacije Bosne i Hercegovine

#### Upravno pravo i javni sektor

- Zakon o upravnom postupku Federacije Bosne i Hercegovine
- Zakon o upravnim sporovima Federacije Bosne i Hercegovine
- Zakon o organizaciji organa uprave u Federaciji Bosne i Hercegovine
- Zakon o inspekcijama Federacije Bosne i Hercegovine
- Zakon o slobodi pristupa informacijama u Federaciji Bosne i Hercegovine
- Zakon o matičnim knjigama Federacije Bosne i Hercegovine
- Zakon o državljanstvu Federacije Bosne i Hercegovine
- Zakon o Poreznoj upravi Federacije Bosne i Hercegovine

#### Privreda, finansije i rad

- Zakon o privrednim društvima Federacije Bosne i Hercegovine
- Zakon o registraciji poslovnih subjekata u Federaciji Bosne i Hercegovine
- Zakon o stečaju Federacije Bosne i Hercegovine
- Zakon o likvidacionom postupku, gdje je zasebno primjenjiv
- Zakon o radu Federacije Bosne i Hercegovine
- Zakon o unutrašnjoj trgovini Federacije Bosne i Hercegovine
- Zakon o obrtu i srodnim djelatnostima u Federaciji Bosne i Hercegovine
- Zakon o računovodstvu i reviziji u Federaciji Bosne i Hercegovine
- Zakon o porezu na dobit Federacije Bosne i Hercegovine
- Zakon o porezu na dohodak Federacije Bosne i Hercegovine
- Zakon o penzijskom i invalidskom osiguranju Federacije Bosne i Hercegovine
- Zakon o mjenici, gdje se primjenjuje u Federaciji Bosne i Hercegovine
- Zakon o čeku, gdje se primjenjuje u Federaciji Bosne i Hercegovine
- Zakon o tržištu vrijednosnih papira Federacije Bosne i Hercegovine
- Zakon o finansijskom poslovanju Federacije Bosne i Hercegovine

#### Pravosudne profesije i posebne oblasti

- Zakon o advokaturi Federacije Bosne i Hercegovine
- Zakon o notarima Federacije Bosne i Hercegovine
- propisi o sudskim vještacima i stalnim sudskim tumačima
- propisi o besplatnoj pravnoj pomoći
- Zakon o zaštiti okoliša Federacije Bosne i Hercegovine
- Zakon o prostornom planiranju i korištenju zemljišta na nivou Federacije Bosne i Hercegovine
- Zakon o poljoprivrednom zemljištu Federacije Bosne i Hercegovine
- Zakon o vodama Federacije Bosne i Hercegovine
- propisi o šumama relevantni za nivo Federacije Bosne i Hercegovine
- Zakon o zdravstvenoj zaštiti Federacije Bosne i Hercegovine
- Zakon o zdravstvenom osiguranju Federacije Bosne i Hercegovine
- propisi o socijalnoj zaštiti relevantni za nivo Federacije Bosne i Hercegovine

### 9.3. Republika Srpska

#### Krivično pravo, pravosuđe i postupci

- Krivični zakonik Republike Srpske
- Zakon o krivičnom postupku Republike Srpske
- Zakon o parničnom postupku Republike Srpske
- Zakon o izvršnom postupku Republike Srpske
- Zakon o opštem upravnom postupku Republike Srpske
- Zakon o upravnim sporovima Republike Srpske
- Zakon o prekršajima Republike Srpske
- Zakon o sudovima Republike Srpske
- Zakon o javnim tužilaštvima Republike Srpske
- Zakon o zaštiti svjedoka u krivičnom postupku Republike Srpske
- Zakon o izvršenju krivičnih i prekršajnih sankcija Republike Srpske

#### Građansko, porodično i imovinsko pravo

- Zakon o stvarnim pravima Republike Srpske
- Zakon o premjeru i katastru Republike Srpske
- Zakon o zemljišnim knjigama Republike Srpske
- Zakon o nasljeđivanju Republike Srpske
- Porodični zakon Republike Srpske
- Zakon o vanparničnom postupku Republike Srpske
- Zakon o obligacionim odnosima koji se primjenjuje u Republici Srpskoj, uz relevantne izmjene i prijelazne izvore
- Zakon o zaštiti od nasilja u porodici Republike Srpske

#### Privreda, finansije i rad

- Zakon o privrednim društvima Republike Srpske
- Zakon o registraciji poslovnih subjekata Republike Srpske
- Zakon o stečaju Republike Srpske
- Zakon o radu Republike Srpske
- Zakon o računovodstvu i reviziji Republike Srpske
- propisi o unutrašnjem platnom prometu relevantni za Republiku Srpsku
- Zakon o porezu na dobit Republike Srpske
- Zakon o porezu na dohodak Republike Srpske
- Zakon o poreskom postupku Republike Srpske
- Zakon o penzijskom i invalidskom osiguranju Republike Srpske
- Zakon o trgovini Republike Srpske
- Zakon o zanatsko-preduzetničkoj djelatnosti Republike Srpske
- Zakon o mjenici, gdje se primjenjuje u Republici Srpskoj
- Zakon o čeku, gdje se primjenjuje u Republici Srpskoj
- Zakon o tržištu hartija od vrijednosti Republike Srpske

#### Uprava i posebne oblasti

- Zakon o republičkoj upravi Republike Srpske
- Zakon o inspekcijama Republike Srpske
- propisi o slobodi pristupa informacijama relevantni za Republiku Srpsku
- Zakon o advokaturi Republike Srpske
- Zakon o notarima Republike Srpske
- Zakon o zaštiti životne sredine Republike Srpske
- Zakon o uređenju prostora i građenju Republike Srpske
- Zakon o poljoprivrednom zemljištu Republike Srpske
- Zakon o zdravstvenoj zaštiti Republike Srpske
- Zakon o zdravstvenom osiguranju Republike Srpske

### 9.4. Brčko distrikt Bosne i Hercegovine

- Krivični zakon Brčko distrikta Bosne i Hercegovine
- Zakon o krivičnom postupku Brčko distrikta Bosne i Hercegovine
- Zakon o parničnom postupku Brčko distrikta Bosne i Hercegovine
- Zakon o izvršnom postupku Brčko distrikta Bosne i Hercegovine
- Zakon o opštem upravnom postupku Brčko distrikta Bosne i Hercegovine
- Zakon o upravnim sporovima Brčko distrikta Bosne i Hercegovine
- Zakon o prekršajima Brčko distrikta Bosne i Hercegovine
- Zakon o sudovima Brčko distrikta Bosne i Hercegovine
- Zakon o Tužilaštvu Brčko distrikta Bosne i Hercegovine
- Zakon o vlasništvu i drugim stvarnim pravima Brčko distrikta Bosne i Hercegovine
- propisi o zemljišnim knjigama Brčko distrikta Bosne i Hercegovine
- Zakon o nasljeđivanju Brčko distrikta Bosne i Hercegovine
- Porodični zakon Brčko distrikta Bosne i Hercegovine
- Zakon o radu Brčko distrikta Bosne i Hercegovine
- Zakon o privrednim društvima Brčko distrikta Bosne i Hercegovine
- propisi o registraciji poslovnih subjekata Brčko distrikta Bosne i Hercegovine
- Zakon o stečaju Brčko distrikta Bosne i Hercegovine
- Zakon o advokaturi Brčko distrikta Bosne i Hercegovine
- propisi o notarima Brčko distrikta Bosne i Hercegovine
- Zakon o javnoj imovini Brčko distrikta Bosne i Hercegovine
- Zakon o Direkciji za finansije Brčko distrikta Bosne i Hercegovine
- propisi o Poreznoj upravi Brčko distrikta Bosne i Hercegovine

### 9.5. Međunarodni i EU korpus relevantan za BiH

- Ustav Bosne i Hercegovine
- Evropska konvencija o ljudskim pravima
- protokoli uz Evropsku konvenciju o ljudskim pravima
- Međunarodni pakt o građanskim i političkim pravima
- Konvencija protiv torture
- Konvencija o pravima djeteta
- Konvencija o eliminaciji svih oblika diskriminacije žena
- Konvencija o pravima osoba s invaliditetom
- Istanbulska konvencija
- Konvencija Ujedinjenih nacija protiv korupcije
- Konvencija Ujedinjenih nacija protiv transnacionalnog organizovanog kriminala
- Konvencija o kibernetičkom kriminalu (Budimpeštanska konvencija)
- Dodatni protokol uz Konvenciju o kibernetičkom kriminalu o kriminalizaciji djela rasističke i ksenofobne prirode počinjenih putem računarskih sistema
- modernizovani okvir Konvencije Vijeća Evrope o zaštiti lica u pogledu automatske obrade ličnih podataka (Konvencija 108+)
- Sporazum o stabilizaciji i pridruživanju
- GDPR
- EU AI Act
- Uredba (EU) 2023/1543 o evropskim nalozima za dostavljanje i nalozima za čuvanje elektronskih dokaza
- Direktiva (EU) 2023/1544 o imenovanju određenih ustanova i pravnih zastupnika radi prikupljanja elektronskih dokaza u krivičnim postupcima
- relevantne EU direktive o procesnim pravima osumnjičenih i optuženih
- Direktiva o pravima žrtava
- relevantne EU direktive o sprečavanju pranja novca

### 9.6. Kantonalni propisi — nakon core korpusa

Kantonalni korpus se ne radi nasumično. Za svaki kanton prvo se unose propisi sa najvišim praktičnim uticajem:

- propisi o sudskim taksama;
- propisi o prostornom uređenju i građenju;
- propisi o imovini kantona;
- propisi o državnoj službi;
- propisi o policijskim službenicima i unutrašnjim poslovima;
- propisi o porezima i lokalnim naknadama;
- propisi o obrazovanju;
- propisi o socijalnoj zaštiti;
- propisi o zdravstvenoj zaštiti i osiguranju, gdje se uređuju na kantonalnom nivou.

## 10. Redoslijed rada nakon postojećih sedam zakona

### Prioritet 1 — zatvaranje pravosudnog core korpusa

1. Krivični zakon Federacije Bosne i Hercegovine
2. Zakon o krivičnom postupku Federacije Bosne i Hercegovine
3. Krivični zakonik Republike Srpske
4. Zakon o krivičnom postupku Republike Srpske
5. Krivični zakon Brčko distrikta Bosne i Hercegovine
6. Zakon o krivičnom postupku Brčko distrikta Bosne i Hercegovine
7. Zakon o izvršnom postupku Brčko distrikta Bosne i Hercegovine
8. Zakon o opštem upravnom postupku Federacije Bosne i Hercegovine
9. Zakon o upravnim sporovima Federacije Bosne i Hercegovine
10. Zakon o opštem upravnom postupku Republike Srpske
11. Zakon o upravnim sporovima Republike Srpske
12. Zakon o opštem upravnom postupku Brčko distrikta Bosne i Hercegovine
13. Zakon o upravnim sporovima Brčko distrikta Bosne i Hercegovine

### Prioritet 2 — građansko, imovinsko, porodično i radno pravo

- stvarna prava, zemljišne knjige, katastar, nasljeđivanje, vanparnični postupak i porodično pravo za FBiH, RS i Brčko;
- rad, PIO, zdravstveno osiguranje, socijalna zaštita i zaštita od nasilja u porodici;
- stečaj, registracija subjekata, mjenica, ček i finansijsko poslovanje.

### Prioritet 3 — pravosuđe, javna uprava, porezi i privreda

- sudovi, tužilaštva, advokatura, notari, vještaci i pravna pomoć;
- porezni postupak, porezne uprave, inspekcije, javne nabavke i pristup informacijama;
- privredna društva, konkurencija, potrošači, komunikacije, elektronski dokumenti i zaštita podataka.

### Prioritet 4 — međunarodno, EU i kantonalno pravo

- međunarodne konvencije i EU instrumenti relevantni za pravosuđe, digitalne dokaze, zaštitu podataka i AI;
- kantonalni propisi po prioritetnim grupama iz odjeljka 9.6.

## 11. Minimalni testovi nakon svake serije zakona

Za svaku seriju importovanih zakona testirati najmanje:

- isti broj člana kroz više jurisdikcija, npr. `član 42`;
- skraćenice zakona, npr. `KZ FBiH`, `KZ RS`, `KZ BD BiH`;
- materijalno/procesno razlikovanje;
- analiziranje presude koja eksplicitno navodi propis i član;
- upit sa nepoznatom jurisdikcijom, gdje sistem mora zatražiti pojašnjenje ili prikazati odvojene režime;
- upit koji se odnosi na historijsku verziju propisa;
- dokaz da retrieval rezultat prikazuje zakon, član, jurisdikciju, status izvora i ograničenje kvaliteta.

## 12. Pravilo za tempo rada

Masovno pripremati zakone, ali ne masovno objavljivati neprovjerene rezultate.

Preporučeni ritam:

```text
batch acquisition + preview: 10–25 zakona po seriji, kada izvori imaju sličan format
canonical transfer: samo READY_FOR_TRANSFER zakoni
production import: 3–5 zakona po izričito odobrenoj seriji
post-import test: obavezan prije naredne serije
```

Ovaj pristup daje brzinu u pripremi korpusa, a sprečava da se kasnije mjesecima ručno popravljaju greške nastale zbog pogrešne jurisdikcije, verzije ili miješanja članova.
