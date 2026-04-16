# Formelbeskrivelse K2 – God oppstalling (en-hest-versjon)

**Gjelder:** `hestevelferd_en_hest.html` (én-hest-versjonen)
**Kategori:** K2 – God oppstalling
**Sub-indekser:** Idx1 (bevegelsesfrihet), Idx2 (liggekomfort inkl. renhet), Idx3 (termisk komfort)
**Sist oppdatert:** 2026-04-16 (runde 1 beslutninger innarbeidet)
**Status:** ✓ Prinsippspørsmål avgjort. Kategori-intern revisjon gjenstår for punkter merket 🔄.

---

## 1. Oversikt over K2

K2 kombinerer tre sub-indekser til én samlet oppstallingsscore (0–100):

| Sub-indeks | Hva måles | Registreringer | Målingstyper |
|---|---|---|---|
| **Idx1** – Bevegelsesfrihet | Mulighet for fri bevegelse og naturlig aktivitet | a) tid på boks 📋, b) scan 🐴, c) ressursplassering 🏠, d) berikelse 🏠 | 1 dyrbasert + 1 styringsbasert + 2 ressursbasert |
| **Idx2** – Liggekomfort | Mulighet for hvile og søvn + renhet | e) strøkvalitet 🏠, f) hestens renhet 🐴 | 1 dyrbasert + 1 ressursbasert |
| **Idx3** – Termisk komfort | Fravær av temperaturstress og tilgang til ly | g) temperaturstress 🐴, h) tilgang til ly 🏠 | 1 dyrbasert + 1 ressursbasert |

**Sammensetning:** Tre-variabel Choquet-integral med kapasiteter fra finsk protokoll:

| Kapasitet | Verdi |
|---|---|
| μ₁ (bevegelse alene) | 0,15 |
| μ₂ (komfort alene) | 0,11 |
| μ₃ (termisk alene) | 0,12 |
| μ₁₂ | 0,34 |
| μ₁₃ | 0,43 |
| μ₂₃ | 0,37 |
| μ₁₂₃ | 1,00 |

**Dekningsgrad:** K2 beregnes hvis minst 2 av 3 sub-indekser er registrert. Manglende sub-indeks fylles med gjennomsnitt av de tilgjengelige. Dekningsgrad vises alltid: f.eks. `K2: 65 (2 av 3)`.

---

## 2. Sub-indeks Idx1 – Bevegelsesfrihet

Idx1 kombinerer bokstid og fribevegelse med finsk vekting:

```
movementScore = round(omaehtoinenScore · 0,6 + boxTimeScore · 0,4)
```

### 2.1 Registrering a – Tid på boks/spilt (`boxTime`) 📋

| Verdi | Beskrivelse | Severity | boxTimeScore |
|---|---|---|---|
| `0` | Kortere enn 14 timer | 0,00 | 100 |
| `1` | 14–18 timer | 0,44 | 56 |
| `2` | Mer enn 18 timer | 1,00 | 0 |

Formel: `boxTimeScore = round(max(0, 100 − severity · 100))`

**Brukerforklaring:** *"Hester trenger tid til fri bevegelse. Under 14 timer på boks er ideelt. Over 18 timer gir kritisk lav score fordi bevegelsesbehovet da er sterkt dempet."*

**Status:** ✓ Severity-skala fra finsk protokoll.

---

### 2.2 Registrering b – Aktivitetsobservasjon (scan) 🐴

**Hva registreres:** 5 observasjoner hvert 4. minutt over 20 minutter i uteområdet. Ved hvert tidspunkt: `'moving'` eller `'standing'`.

```
standing = antall observasjoner satt til 'standing' (0–5)
scanScore = round(max(0, 100 − standing · 20))
```

| Antall "står" | scanScore |
|---|---|
| 0 av 5 | 100 |
| 1 av 5 | 80 |
| 2 av 5 | 60 |
| 3 av 5 | 40 |
| 4 av 5 | 20 |
| 5 av 5 | 0 |

**Foreslått beslutning – plassering av scan:** Scan flyttes fra Idx2 (liggekomfort) til Idx1 (bevegelsesfrihet) i beregningen, slik at den matcher plasseringen i skjemaet. Begrunnelse: observasjonen måler aktivitet utendørs midt på dagen, ikke liggeatferd om natten. At en hest står stille ute sier noe om bevegelsesmuligheten, ikke om strøkvaliteten.

**Implementering av foreslått endring:** ScanScore inngår i `omaehtoinenScore`-beregningen. Foreslått formel:

```
omaehtoinenBase = beslutningstabell(resources, enrichment)    (som i dag)
omaehtoinenScore = round(omaehtoinenBase · 0,6 + scanScore · 0,4)
movementScore = round(omaehtoinenScore · 0,6 + boxTimeScore · 0,4)
```

**Brukerforklaring:** *"Scan-observasjonen viser om hesten faktisk beveger seg når den har muligheten. En hest som bare står i 20 minutter kan ha for lite stimulering i uteområdet."*

**Status:** 🔄 Krever endring i `calcK2()`.

---

### 2.3 Registrering c – Ressursplassering (`resources`) 🏠

| Verdi | Beskrivelse |
|---|---|
| `yes` | Ressursene er spredt (minst 20 m fra hverandre) |
| `no` | Ressursene er samlet |

**Brukerforklaring:** *"Når vann, fôr og ly er spredt, må hesten gå for å dekke behovene sine. Det fremmer naturlig bevegelse."*

---

### 2.4 Registrering d – Berikelse i uteområdet 🏠

To kategorier med checkboxer:
- **Matsøk:** Slowfeeder/høynett, grener/kvister, gress i paddocken
- **Bevegelse:** Bakke/gang/korridor, trær/skog, kløbørste/stein/vannspreder

Pluss "Ingen av disse" i hver kategori.

**Avledede flagg:** `enriched` (begge kategorier dekket), `hasAnyEnrich` (noe avkrysset), `enrichAnswered` (noe valgt, inkl. "ingen").

---

### 2.5 Beslutningstabell – ressurser + berikelse → `omaehtoinenBase`

| Ressurser spredt? | Berikelse | `omaehtoinenBase` |
|---|---|---|
| `yes` | begge kategorier | 100 |
| `yes` | delvis | 90 |
| `yes` | ingen / "ingen av disse" | 80 |
| `yes` | ikke besvart | 90 |
| `no` | begge kategorier | 70 |
| `no` | delvis | 65 |
| `no` | ingen | 60 |
| `no` | ikke besvart | 65 |
| ikke besvart | begge | 85 |
| ikke besvart | delvis | 75 |
| ikke besvart | ingen | 65 |

**Brukerforklaring:** *"Et uteområde med spredte ressurser og berikelse i begge kategorier gir best score. Samlede ressurser uten berikelse gir trekk – hesten har da liten grunn til å bevege seg."*

**Status:** ✓ Kalibrert mot finsk kode. Runde tall er en forenkling fra finsk beslutningstre.

---

## 3. Sub-indeks Idx2 – Liggekomfort (inkl. renhet)

Etter foreslått endring (scan flyttes til Idx1) består Idx2 av strøkvalitet og renhet:

```
comfortScore = round(beddingSplineScore · 0,7 + cleanScore · 0,3)
```

### 3.1 Registrering e – Strøkvalitet (`bedding`) 🏠

| Verdi | Beskrivelse | Severity | beddingI |
|---|---|---|---|
| `0` | Optimalt (≥ 10 cm, eller 5 cm + matte, tørt) | 0,00 | 100 |
| `1` | Akseptabelt (5–10 cm uten matte, tørt) | 0,44 | 56 |
| `2` | Ikke godkjent (vått eller < 5 cm) | 1,00 | 0 |
| `na` | Helårs utegang | – | (utelates) |

COMFORT-splinen (knot 62) konverterer beddingI til score:
- Klasse 0 → score 100, klasse 1 → score ≈ 43, klasse 2 → score 0.

**Brukerforklaring:** *"Hesten trenger tørt, dypt strø for REM-søvn. Uten god liggeflate vil hesten ikke legge seg ned, og det påvirker hvile og restitusjon."*

**Status:** ✓ WQ-spline verifisert.

---

### 3.2 Registrering f – Hestens renhet (`clean`) 🐴

**Foreslått gradering (fra binær til 0/1/2):**

| Verdi | Beskrivelse | cleanScore |
|---|---|---|
| `0` | Ren | 100 |
| `1` | Lett tilsmusset (noe skitt, men ikke fastsittende) | 70 |
| `2` | Fastsittende skitt (inntørket, > kredittkort) | 30 |

**Endring fra forrige versjon:** Mellomkategori lagt til for bedre oppløsning. I dag er det bare 100/30 som gir et stort hopp.

**Brukerforklaring:** *"En skitten hest er et tegn på at miljøet er dårlig. Fastsittende, inntørket skitt tyder på langvarig kontakt med vått, skittent underlag."*

**Status:** 🔄 Krever HTML-endring (nytt radioknapp-valg).

---

### 3.3 Aggregering til `comfortScore`

```
comfortScore = round(beddingSplineScore · 0,7 + cleanScore · 0,3)
```

Hvis bare én er tilgjengelig, brukes den alene. Vekting 70/30 følger finsk protokoll.

---

## 4. Sub-indeks Idx3 – Termisk komfort

### 4.1 Registrering g – Tegn på temperaturstress (`thermal`) 🐴

**Foreslått gradering (fra binær til 0/1/2):**

| Verdi | Beskrivelse | thermalStress |
|---|---|---|
| `0` | Ingen tegn observert | 100 |
| `1` | Milde tegn (lett urolig, noe svette uten anstrengelse) | 50 |
| `2` | Tydelig stress (skjelving, kraftig svetting, anstrengt pust) | 0 |

**Endring fra forrige versjon:** I dag mapper alle stress-verdier (shiver/sweat/pant) til 0. Med gradering gis rom for milde tegn uten å utløse nullscore. Assessor trenger å skille mellom "hesten virker litt ukomfortabel" og "hesten har tydelig temperaturstress".

**Brukerforklaring:** *"Skjelving (kulde) og kraftig svetting/anstrengt pust (varme) er alvorlige tegn. Milde tegn gir noe trekk, fordi det indikerer at miljøet ikke er helt riktig."*

**Status:** 🔄 Krever HTML-endring (erstatte fire spesifikke valg med tre generelle alvorlighetsnivåer).

---

### 4.2 Registrering h – Tilgang til ly (`shelter`) 🏠

**Foreslått gradering (fra binær til 0/1/2):**

| Verdi | Beskrivelse | shelterScore |
|---|---|---|
| `0` | Godkjent ly (leskur med tak + 3 vegger, tett skog) | 100 |
| `1` | Delvis ly (takutspring, glissen skog, vindskjerm uten tak) | 50 |
| `2` | Ingen ly | 0 |

**Endring fra forrige versjon:** Mellomkategori gir rom for "det finnes noe ly, men ikke ideelt". I dag er det binært (100/0).

**Brukerforklaring:** *"Hester trenger ly mot vind, regn og sol. Godkjent ly gir full score. Delvis ly er bedre enn ingenting, men ikke tilstrekkelig i alle værforhold."*

**Status:** 🔄 Krever HTML-endring.

---

### 4.3 Aggregering til `thermalScore`

```
thermalScore = round(thermalStress · 0,6 + shelterScore · 0,4)
```

Finsk vekting: observert stress (utfall 🐴) teller mer enn strukturell tilgang (ressurs 🏠). En hest uten ly men uten observert stress får thermalScore = 100·0,6 + 0·0,4 = 60. En hest med mild stress og godkjent ly får 50·0,6 + 100·0,4 = 70.

---

## 5. Aggregering til K2 – Choquet-integral

### 5.1 Håndtering av manglende sub-indekser

Følger oversiktsdokumentets beslutning #3:
- Minimum 2 av 3 sub-indekser for å vise numerisk score
- Manglende fylles med gjennomsnitt av tilgjengelige
- Dekningsgrad vises alltid

### 5.2 Choquet-algoritmen

Sub-indeksene sorteres stigende. For hver differens brukes kapasiteten til "gjenværende mengde".

**Eksempel:** movementScore = 40, comfortScore = 70, thermalScore = 90
- K2 = 40·μ₁₂₃ + (70−40)·μ₂₃ + (90−70)·μ₃ = 40·1,00 + 30·0,37 + 20·0,12 = **53**

**Brukerforklaring:** *"Oppstallings-scoren styres av det svakeste området. Hvis bevegelsesfriheten er dårlig, hjelper det lite at strø og ly er perfekt. Alle tre må være på plass."*

---

## 6. Oppsummeringstabell

| # | Registrering | Type | Sub-indeks | Maks positiv | Maks negativ |
|---|---|---|---|---|---|
| a | Tid på boks | 📋 | Idx1 | 100 (< 14 t) | 0 (> 18 t) |
| b | Scan (aktivitet 20 min) | 🐴 | Idx1 (etter endring) | 100 (0/5 standing) | 0 (5/5 standing) |
| c | Ressursplassering | 🏠 | Idx1 | 100 (spredt + beriket) | 60 (samlet + ingen) |
| d | Berikelse | 🏠 | Idx1 | Se tabell 2.5 | Se tabell 2.5 |
| e | Strøkvalitet | 🏠 | Idx2 | 100 (optimalt) | 0 (vått/tynt) |
| f | Hestens renhet | 🐴 | Idx2 | 100 (ren) | 30 (fastsittende) |
| g | Temperaturstress | 🐴 | Idx3 | 100 (ingen tegn) | 0 (tydelig stress) |
| h | Tilgang til ly | 🏠 | Idx3 | 100 (godkjent) | 0 (ingen) |

---

## 7. Gjenstående åpne spørsmål (runde 2)

| # | Spørsmål | Foreslått løsning | Status |
|---|---|---|---|
| 1 | Scan – plassering (skjema vs. kode) | Flytt til Idx1 i beregning (matcher skjema) | 🔄 Foreslått |
| 2 | Thermal – fire spesifikke stress-typer → tre generelle | Gradér 0/1/2 som beskrevet | 🔄 Foreslått |
| 3 | Shelter – gradér fra binær | Legg til "delvis ly" mellomkategori | 🔄 Foreslått |
| 4 | Clean – gradér fra binær | Legg til "lett tilsmusset" mellomkategori | 🔄 Foreslått |
| 5 | Berikelse-tabell med runde tall (60–100) | Beholder for nå. Kan eventuelt erstattes med spline i fremtiden. | ✓ Foreslått |

---

## 8. Referanser i kode

| Element | Linjenummer (ca.) |
|---|---|
| `S`-state initialisering (K2-felter) | 1050–1051 |
| `setScan()` | 1203–1221 |
| `handleEnrichment()` | 1223–1235 |
| `SPLINE.COMFORT` | 1322–1327 |
| `CHOQUET_CAPS.K2` | 1346–1350 |
| `calcK2()` | 1515–1661 |

HTML-inputfeltene: `<div id="page-k2">` rundt linje 584–710.
