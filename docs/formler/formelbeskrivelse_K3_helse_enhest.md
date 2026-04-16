# Formelbeskrivelse K3 – God helse (en-hest-versjon)

**Gjelder:** `hestevelferd_en_hest.html` (én-hest-versjonen)
**Kategori:** K3 – God helse
**Sub-indekser:** Idx1 (skader), Idx2 (annen smerte), Idx3 (sykdom), Idx4 (smerte fra bruk)
**Sist oppdatert:** 2026-04-16 (runde 1 beslutninger innarbeidet)
**Status:** 🔄 Mest omfattende endringer i denne kategorien – mange binære målinger graderes og SKIN-spline aktiveres.

---

## 1. Oversikt over K3

| Sub-indeks | Hva måles | Registreringer (med type) |
|---|---|---|
| **Idx1** – Fravær av skader | Fysiske skader | skin 🐴, swelling 🐴, hoof 🐴, scratch 🐴, lameness 🐴 |
| **Idx2** – Fravær av annen smerte | Smerte som ikke skyldes bruk/skade | HGS (6 trekk) 🐴, back 🐴 |
| **Idx3** – Fravær av sykdom | Symptomer på sykdom | nasal 🐴, eye 🐴, cough 🐴, diarrhoea 🐴, colic 📋 |
| **Idx4** – Smerte fra bruk | Utstyr- og håndteringsrelatert | mouth 🐴, tack 🐴, saddling 🐴 |

**Merk:** K3 er den mest dyrebaserte kategorien (🐴 dominerer). Kun kolikk (historisk opplysning) er styringsbasert (📋).

**Choquet-kapasiteter (finsk Taulukko 18):**

| μ | Verdi | Rangering |
|---|---|---|
| μ₁ (skader) | 0,08 | Lavest |
| μ₂ (annen smerte) | **0,21** | **Høyest** |
| μ₃ (sykdom) | 0,12 | |
| μ₄ (smerte fra bruk) | 0,17 | |
| μ₁₂ | 0,28 | |
| μ₁₃ | 0,21 | |
| μ₁₄ | 0,25 | |
| μ₂₃ | 0,32 | |
| μ₂₄ | 0,39 | |
| μ₃₄ | 0,28 | |
| μ₁₂₃ | 0,42 | |
| μ₁₂₄ | 0,48 | |
| μ₁₃₄ | 0,44 | |
| μ₂₃₄ | 0,52 | |
| μ₁₂₃₄ | 1,00 | |

**Dekningsgrad:** K3 beregnes hvis minst 3 av 4 sub-indekser er registrert. Manglende fylles med gjennomsnitt. Vises: `K3: 72 (3 av 4)`.

**Brukerforklaring K3:** *"Helsescoren fanger smerte, skader og sykdom. Systemet er designet slik at pågående smerte (HGS, rygg, brukssmerte) veier tyngst – fordi en hest som har vondt har dårlig velferd, selv om alt annet ser bra ut."*

---

## 2. Sub-indeks Idx1 – Fravær av skader

### 2.1 Integument-gruppen (skin + swelling + hoof + scratch)

Alle fire graderes 0/1/2 og bruker SKIN-spline med **finske severity-vekter (mild = 0,2, alvorlig = 1,0)**. Disse vektene er lavere enn standard (0,5 / 1,0) fordi finsk protokoll differensierer: milde hudskader anses som mindre alvorlige enn milde sykdomsfunn.

**Severity-formel for integument (etter revisjon):**

```
For hvert integument-funn med N = 1:
    virtualPrev = mild · 0,2 + alvorlig · 1,0
    I = max(0, 100 − virtualPrev · 100)
    score = WQ-spline(I, SPLINE.SKIN)
```

#### Registrering – Hudlesjoner (`skin`) 🐴

| Verdi | Beskrivelse | virtualPrev | I | Score (ca.) |
|---|---|---|---|---|
| 0 | Ingen skader | 0 | 100 | 100 |
| 1 | Mild (overfladisk, eldre, < kredittkort) | 0,2 | 80 | 57 |
| 2 | Alvorlig (dype, ferske, åpne, > kredittkort) | 1,0 | 0 | 0 |

#### Registrering – Hevelse (`swelling`) 🐴

| Verdi | Beskrivelse | virtualPrev | I | Score (ca.) |
|---|---|---|---|---|
| 0 | Ingen | 0 | 100 | 100 |
| 1 | Kald (kronisk, vindgaller) | 0,2 | 80 | 57 |
| 2 | Varm (akutt betennelse) | 1,0 | 0 | 0 |

#### Registrering – Hovproblemer (`hoof`) 🐴

**Gradert fra binær (runde 1 beslutning #5):**

| Verdi | Beskrivelse | virtualPrev | I | Score (ca.) |
|---|---|---|---|---|
| 0 | Ingen hovproblemer | 0 | 100 | 100 |
| 1 | Mild (én hov, lette sprekker) | 0,2 | 80 | 57 |
| 2 | Alvorlig (flere hover, byll, overgrodd) | 1,0 | 0 | 0 |

#### Registrering – Gnagemerker (`scratch`) 🐴

**Gradert fra binær:**

| Verdi | Beskrivelse | virtualPrev | I | Score (ca.) |
|---|---|---|---|---|
| 0 | Ingen | 0 | 100 | 100 |
| 1 | Lett (noe tynnslitt hår) | 0,2 | 80 | 57 |
| 2 | Tydelig (avgnagd man, bare flekker) | 1,0 | 0 | 0 |

#### Aggregering av integument til `integScore`

De fire SKIN-spline-scorene aggregeres med weighted min-max (worst = 0,70):

```
integScore = round(worst · 0,70 + gjennomsnitt(rest) · 0,30)
```

**Brukerforklaring:** *"Vi ser på fire typer skader. Den alvorligste dominerer scoren – én alvorlig skade trekker mer enn mange milde."*

---

### 2.2 Halthet (`lameness`) 🐴

**Gradert fra binær:**

| Verdi | Beskrivelse |
|---|---|
| 0 | Ingen synlig halthet |
| 1 | Mild (kun synlig i trav, ikke i skritt) |
| 2 | Alvorlig (synlig i skritt, eller tydelig nikking/vipping) |

**⚠ LAMENESS-spline er problematisk for N = 1.** Splinen er svært flat i starten – den er kalibrert for flokk-prevalens der 5–10 % halte er "litt" og 50 % er "alvorlig". For N = 1 gir mild halthet (virtualPrev = 0,5) en score på ca. 9, som er urimelig strengt.

**Foreslått beslutning:** Bruk direkte severity-mapping for lameness inntil LAMENESS-spline er kalibrert for N = 1 av Essi:

| Verdi | lameScore |
|---|---|
| 0 | 100 |
| 1 (mild) | 50 |
| 2 (alvorlig) | 0 |

Disse verdiene er valgt for å gi en meningsfull progresjon der mild halthet er et tydelig funn (halvparten av full score tapt) men ikke katastrofalt, mens alvorlig halthet gir nullscore – en halt hest har per definisjon svekket velferd.

**Brukerforklaring:** *"Halthet er alltid alvorlig, men det er forskjell på en hest som er litt ujevn i trav og en som er tydelig halt i skritt."*

---

### 2.3 Aggregering av integument + halthet til `injuryScore`

Todelt Choquet med finske kapasiteter:

```
mn = min(integScore, lameScore)
mx = max(integScore, lameScore)
μ = 0,56 hvis integScore ≥ lameScore
    0,31 hvis lameScore > integScore
injuryScore = round(mn · 1,0 + (mx − mn) · μ)
```

**Status:** ✓ μ-verdiene fra finsk protokoll.

---

## 3. Sub-indeks Idx2 – Fravær av annen smerte

Idx2 har **høyeste μ alene (0,21)** i K3-Choquet. Strengeste aggregering: `min()`.

### 3.1 HGS (Horse Grimace Scale) 🐴

6 ansiktstrekk (A–F), hver 0/1/2. Sum 0–12. Krever minst 3 registrerte trekk.

**Poengberegning:**

```
hgsI = max(0, 100 − (hgsSum / 12) · 100)
```

| hgsSum | hgsI | Kategori |
|---|---|---|
| 0 | 100 | 🟢 Normal |
| 3 | 75 | 🟢 Lav/normal |
| 6 | 50 | 🟡 Moderat |
| 9 | 25 | 🔴 Alvorlig |
| 12 | 0 | 🔴 Svært alvorlig |

**Brukerforklaring:** *"HGS vurderer smerte via hestens ansiktsuttrykk i ro. Scoren synker gradvis med økende smertetegn. HGS er mest pålitelig når hesten er uforstyrret og ikke spiser."*

**Åpent spørsmål:** HGS bruker lineær skalering, mens andre målinger bruker WQ-spline. Bør konverteres til virtualPrev-basert spline for konsistens hvis Essi har en HGS-spesifikk spline.

---

### 3.2 Ryggsmerte (`back`) 🐴

**Gradert fra binær:**

| Verdi | Beskrivelse | backScore |
|---|---|---|
| 0 | Ingen reaksjon | 100 |
| 1 | Mild respons (ett punkt, viker unna) | 50 |
| 2 | Kraftig respons (flere steder, snapper, slår) | 0 |

**Brukerforklaring:** *"Ryggsmerte oppdages ved forsiktig trykk langs ryggsøylen. En hest som reagerer kraftig har sannsynligvis kroniske smerter."*

---

### 3.3 Aggregering til `otherPainScore`

```
otherPainScore = min(hgsScore, backScore)
```

Strengeste aggregering. Begrunnelse: enhver indikasjon på smerte skal slå igjennom; "god HGS" skal ikke kompensere for "vondt i ryggen".

**Status:** ✓ I tråd med finsk protokoll.

---

## 4. Sub-indeks Idx3 – Fravær av sykdom

5 funn med differensiert alarm-verdi → kumulativ alarm-sum → DISEASE-spline.

### 4.1 De fem funnene

| Registrering | Type | Verdi | Alarm |
|---|---|---|---|
| Neseflod (`nasal`) | 🐴 | 0 | 0 |
| | | 1 (klar, > 3 cm) | 0,5 |
| | | 2 (tykk/uklar) | 1,0 |
| Øyeflod (`eye`) | 🐴 | 0 | 0 |
| | | 1 (klar > 3 cm) | 0,5 |
| | | 2 (tykk, skorper) | 1,0 |
| Hoste (`cough`) | 🐴 | 0 | 0 |
| | | 1 (sporadisk) | 0,5 |
| | | 2 (gjentakende) | 1,0 |
| Diaré (`diarrhoea`) | 🐴 | 0 | 0 |
| | | 1 (løs avføring) | 0,75 |
| | | 2 (tilsølt bakpart) | 1,5 |
| Kolikk 12 mnd (`colic`) | 📋 | 0 | 0 |
| | | 1 (én episode) | 1,5 |
| | | 2 (behandlet/gjentatt) | 3,0 |

**Endringer:** Hoste og diaré gradert fra binær. Kolikk gradert fra binær. Maks alarmsum justert.

**Foreslått beslutning – maxAlarms:** Endres fra 7 til **7,5** for å matche faktisk maks (1,0 + 1,0 + 1,0 + 1,5 + 3,0).

### 4.2 Poengberegning

```
diseaseI = max(0, 100 − (diseaseAlarms / 7,5) · 100)
diseaseScore = round(WQ-spline(diseaseI, SPLINE.DISEASE))
```

**Referanseverdier:**

| Alarmsum | diseaseI | diseaseScore (ca.) |
|---|---|---|
| 0 | 100 | 100 |
| 1 | 87 | 90 |
| 2 | 73 | 68 |
| 3 | 60 | 46 |
| 4 | 47 | 31 |
| 5 | 33 | 20 |
| 7,5 | 0 | 0 |

**Brukerforklaring:** *"Sykdomstegnene summeres med ulik vekt. Hoste og neseflod er vanlige og gir moderat trekk. Kolikk er potensielt livstruende og gir mye mer."*

---

## 5. Sub-indeks Idx4 – Smerte fra bruk

### 5.1 Munnfunn (`mouth`) 🐴

| Verdi | Beskrivelse | Score |
|---|---|---|
| 0 | Ingen funn | 100 |
| 1 | Mild (pigmentforandring, gamle arr) | 40 |
| 2 | Alvorlig (ferske sår, rødhet, keratinisering) | 5 |
| `na` | Ikke aktuelt | (utelates) |

### 5.2 Utstyrsskader (`tack`) 🐴

**Gradert fra binær:**

| Verdi | Beskrivelse | Score |
|---|---|---|
| 0 | Ingen | 100 |
| 1 | Mild (gnag uten åpent sår) | 60 |
| 2 | Alvorlig (åpent sår fra utstyr) | 10 |

### 5.3 Påsalingsatferd (`saddling`) 🐴

12 atferder observeres under påsaling. Klassifiseres 0/1/2 basert på antall og apati.

| Klasse | Betingelse | Score |
|---|---|---|
| 0 | ≤ 2 atferder uten apati | 100 |
| 1 | 3–6 atferder, eller ≤ 2 + apati | 45 |
| 2 | > 6 atferder, eller > 2 + apati | 5 |

Hvis `saddlingRelevant = 'no'`: utelates.

### 5.4 Aggregering til `mgmtPainScore`

**Foreslått forenkling:** Erstatt dagens kompliserte vektformel med weighted min-max (0,70):

```
mgmtPainScore = round(worst · 0,70 + gjennomsnitt(rest) · 0,30)
```

**Endring fra forrige versjon:** Fjerner intern vekting (2,0 / 1,0 / 1,5) og den ad-hoc-formelen. Bruker i stedet konsistent worst-dominance som i integument og vann. Munnfunn er fortsatt tyngst i praksis fordi den har lavest score ved funn (5 ved alvorlig), ikke fordi den har en spesiell vekt.

**Brukerforklaring:** *"Brukssmerte handler om hesten er påført smerte gjennom utstyr eller håndtering. Munnundersøkelsen veier tyngst fordi munnfunn er direkte tegn på smerte fra bitt eller tannproblemer."*

---

## 6. Aggregering til K3 – Choquet-integral

### 6.1 Håndtering av manglende sub-indekser

Minimum 3 av 4. Manglende fylles med gjennomsnitt. Dekningsgrad vises.

### 6.2 Eksempel

`injuryScore = 80 (idx 1), otherPainScore = 40 (idx 2), diseaseScore = 90 (idx 3), mgmtPainScore = 70 (idx 4)`

Sortert: 40 (idx 2), 70 (idx 4), 80 (idx 1), 90 (idx 3)
- 40 · μ₁₂₃₄ + 30 · μ₁₃₄ + 10 · μ₁₃ + 10 · μ₃
- = 40·1,00 + 30·0,44 + 10·0,21 + 10·0,12 = 40 + 13,2 + 2,1 + 1,2 = **56**

**Brukerforklaring:** *"Helsescoren styres av det svakeste området. En hest med god hud og gode lunger men tydelig smerte (HGS/rygg) får lav score – smerte skal alltid være synlig i resultatet."*

---

## 7. Oppsummeringstabell

| # | Registrering | Type | Sub-indeks | Scoring | Maks neg |
|---|---|---|---|---|---|
| 1 | Hudlesjoner (0/1/2) | 🐴 | Idx1 integ | SKIN-spline (0,2/1,0) | 0 |
| 2 | Hevelse (0/1/2) | 🐴 | Idx1 integ | SKIN-spline (0,2/1,0) | 0 |
| 3 | Hovproblemer (0/1/2) | 🐴 | Idx1 integ | SKIN-spline (0,2/1,0) | 0 |
| 4 | Gnagemerker (0/1/2) | 🐴 | Idx1 integ | SKIN-spline (0,2/1,0) | 0 |
| 5 | Halthet (0/1/2) | 🐴 | Idx1 lame | Direkte (100/50/0) | 0 |
| 6 | HGS-sum (0–12) | 🐴 | Idx2 | Lineær | 0 |
| 7 | Ryggsmerte (0/1/2) | 🐴 | Idx2 | Direkte (100/50/0) | 0 |
| 8 | Neseflod (0/1/2) | 🐴 | Idx3 alarm | 0 / 0,5 / 1,0 | alarm |
| 9 | Øyeflod (0/1/2) | 🐴 | Idx3 alarm | 0 / 0,5 / 1,0 | alarm |
| 10 | Hoste (0/1/2) | 🐴 | Idx3 alarm | 0 / 0,5 / 1,0 | alarm |
| 11 | Diaré (0/1/2) | 🐴 | Idx3 alarm | 0 / 0,75 / 1,5 | alarm |
| 12 | Kolikk 12 mnd (0/1/2) | 📋 | Idx3 alarm | 0 / 1,5 / 3,0 | alarm |
| 13 | Munnfunn (0/1/2/na) | 🐴 | Idx4 | Direkte (100/40/5) | 5 |
| 14 | Utstyrsskader (0/1/2) | 🐴 | Idx4 | Direkte (100/60/10) | 10 |
| 15 | Påsaling (0/1/2) | 🐴 | Idx4 | Direkte (100/45/5) | 5 |

---

## 8. Gjenstående åpne spørsmål (runde 2)

| # | Spørsmål | Foreslått løsning | Status |
|---|---|---|---|
| 1 | LAMENESS-spline urimelig for N=1 | Direkte mapping (100/50/0) inntil kalibrering med Essi | ✓ Foreslått |
| 2 | HGS lineær vs. spline | Beholder lineær, flagg for Essi om det finnes HGS-spline | 🔄 Essi |
| 3 | Kolikk 3,0 alarm – for mye for historisk funn? | Beholder, men gradering (0/1,5/3,0) demper effekten for milde tilfeller | ✓ Foreslått |
| 4 | maxAlarms 7 → 7,5 | Endres | ✓ Foreslått |
| 5 | mgmtPainScore – forenkling | Weighted min-max (0,70) som integ/vann | ✓ Foreslått |
| 6 | SKIN severity 0,2/1,0 vs 0,5/1,0 | Bruk 0,2/1,0 (finsk kalibrering) | ✓ Foreslått |

---

## 9. Referanser i kode

| Element | Linjenummer (ca.) |
|---|---|
| `S`-state (K3-felter) | 1053–1055 |
| `updateSaddling()` | 1238–1255 |
| `updateHGS()` | 1258–1270 |
| `SPLINE.SKIN` | 1304–1309 |
| `SPLINE.LAMENESS` | 1310–1315 |
| `SPLINE.DISEASE` | 1317–1321 |
| `CHOQUET_CAPS.K3` | 1353–1358 |
| `calcK3()` | 1664–1822 |

HTML: `<div id="page-k3">` rundt linje 713–886.
