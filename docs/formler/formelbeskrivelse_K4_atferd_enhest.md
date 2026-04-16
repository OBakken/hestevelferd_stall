# Formelbeskrivelse K4 – Hensiktsmessig atferd (en-hest-versjon)

**Gjelder:** `hestevelferd_en_hest.html` (én-hest-versjonen)
**Kategori:** K4 – Hensiktsmessig atferd
**Sub-indekser:** Idx1 (sosial atferd), Idx2 (naturlig atferd/beite), Idx3 (menneske-hest), Idx4 (emosjonell tilstand)
**Sist oppdatert:** 2026-04-16 (runde 1 beslutninger innarbeidet)
**Status:** ✓ Prinsippspørsmål avgjort. Få binære målinger i K4 – de fleste er allerede graderte.

---

## 1. Oversikt over K4

| Sub-indeks | Hva måles | Registreringer (med type) |
|---|---|---|
| **Idx1** – Sosial atferd | Mulighet for og kvalitet på sosial kontakt | scenario 🏠, indoorContact 🏠, sosial obs 🐴 |
| **Idx2** – Naturlig atferd | Tid med tilgang til beite | pastureDays 📋, enrichDays+enrichHours 📋 |
| **Idx3** – Menneske-hest | Resultat av tilnærmingstest | tester 📋, vaaResult 🐴 |
| **Idx4** – Emosjonell tilstand | Forekomst av stereotypier og frustrasjon | obsMinutes 📋, frustration 🐴, stereotype 🐴 |

**Choquet-kapasiteter (finsk protokoll):**

| μ | Verdi |
|---|---|
| μ₁ (sosial) | 0,10 |
| μ₂ (beite) | **0,07** (lavest) |
| μ₃ (menneske) | 0,12 |
| μ₄ (emosjonell) | **0,17** (høyest) |
| μ₁₂ | 0,12 |
| μ₁₃ | 0,12 |
| μ₁₄ | 0,18 |
| μ₂₃ | 0,15 |
| μ₂₄ | 0,19 |
| μ₃₄ | 0,27 |
| μ₁₂₃ | 0,42 |
| μ₁₂₄ | 0,49 |
| μ₁₃₄ | 0,52 |
| μ₂₃₄ | 0,48 |
| μ₁₂₃₄ | 1,00 |

**Rangering:** emosjonell (0,17) > menneske (0,12) > sosial (0,10) > beite (0,07).

**Åpent spørsmål for runde 3:** μ₂ (beite) = 0,07 er lavest av alle enkelt-μ i hele skjemaet. For norske forhold virker dette kontraintuitivt – beitetid er ofte den viktigste enkeltfaktoren i norsk hestehold. Beholdes som finsk verdi for nå, men flagges for diskusjon med Essi. Det kan tenkes at finsk protokoll vekter beite lavt fordi det er en resource/management-indikator, og emosjonell tilstand (som fanges av stereotypi-observasjon) er en animal-based indikator som fanger *effekten* av manglende beite.

**Dekningsgrad:** K4 beregnes hvis minst 3 av 4 sub-indekser er registrert. Manglende fylles med gjennomsnitt. Vises: `K4: 58 (3 av 4)`.

**Brukerforklaring K4:** *"Atferdsscoren vurderer om hesten kan leve ut naturlig atferd: sosial kontakt, beiting, trygg menneske-kontakt, og fravær av stress-tegn. Stereotypier og frustrasjon veier tyngst fordi de er direkte tegn på at hesten ikke har det bra – uansett hvor gode forholdene ser ut."*

---

## 2. Sub-indeks Idx1 – Sosial atferd

Bygges i tre lag: grunnscore fra scenario, indoor-bonus, og kvalitetsmodifikator.

### 2.1 Scenario (`scenario`) 🏠

| Verdi | Beskrivelse | Grunnscore |
|---|---|---|
| 0 | Sammen med andre (flokk/par) | 100 |
| 1 | Alene, mule-til-mule uten strøm | 70 |
| 2 | Alene, mule-til-mule med strøm | 40 |
| 3 | Alene, ingen fysisk kontakt mulig | 5 |

**Brukerforklaring:** *"Hester er flokkdyr. Full sosial kontakt gir best score. Elektrisk gjerde mellom hester hindrer normal sosial berøring og gir tydelig trekk."*

### 2.2 Indoor contact (`indoorContact`) 🏠

| Verdi | Effekt |
|---|---|
| `yes` | +15 bonus (kun scenario 1–3) |
| `no` | Ingen effekt |
| `na` | Ingen effekt |

Hvis kun indoorContact uten scenario: `yes` → 60, `no` → 30.

### 2.3 Sosial observasjon (kvalitetsmodifikator) 🐴

Aktiveres kun hvis `socialPossible === 'yes'` og hesten ble observert med andre. Tre tellinger:
- `positive` (hvile sammen, pelspleie)
- `negMild` (truende ører, driver vekk)
- `negStrong` (biter, sparker, jager)

```
qualityBalance = positive − negMild − negStrong · 3
modifier = round(max(−20, min(20, (qualityBalance / totalInteractions) · 20)))
socialScore = max(0, min(100, grunnscore + indoorBonus + modifier))
```

**Brukerforklaring:** *"Modifikatoren viser kvaliteten på sosial kontakt. Positiv interaksjon (gjensidig pelspleie, hvile sammen) løfter scoren. Aggresjon trekker ned, og kraftig aggresjon (spark, bitt) trekker tre ganger så mye som mild."*

---

## 3. Sub-indeks Idx2 – Naturlig atferd (beite)

### 3.1 Registreringer 📋

- `pastureDays`: Beitedager per år (≥ 6 t/dag)
- `enrichDays` + `enrichHours`: Kortere beiteperioder (< 6 t), dager per år og timer per dag

### 3.2 Poengberegning

```
fullDays = pastureDays eller 0
enrichFraction = enrichDays · min(enrichHours, 5) / 6
totalEffectiveDays = min(365, fullDays + enrichFraction)
pastureI = (totalEffectiveDays / 365) · 100
pastureScore = round(WQ-spline(pastureI, SPLINE.PASTURE))
```

**Referanseverdier:**

| totalEffectiveDays | pastureI | pastureScore |
|---|---|---|
| 0 | 0 | 0 |
| 90 (3 mnd) | 25 | 42 |
| 183 (6 mnd) | 50 | 77 |
| 270 (9 mnd) | 74 | 95 |
| 365 | 100 | 100 |

**Brukerforklaring:** *"Beiting er den mest naturlige aktiviteten for hester. Scoren øker raskt opp til 6 måneder, deretter flater den ut. Kort beiteperioder (under 6 timer) teller delvis – en time beite om dagen gir ikke null, men heller ikke like mye som en hel beitedag."*

**Status:** ✓ PASTURE-spline fra finsk protokoll.

---

## 4. Sub-indeks Idx3 – Menneske-hest-relasjon

### 4.1 Tilnærmingstest (`vaaResult`) 🐴

| Verdi | Beskrivelse | humanI |
|---|---|---|
| `vaa_plus` | Frivillig positiv (hesten kommer) | 100 |
| `faa_plus` | Tvungen positiv (aksepterer berøring) | 75 |
| `faa_passive` | Tvungen passiv (ingen reaksjon) | 50 |
| `vaa_minus` | Frivillig negativ (trekker seg) | 25 |
| `faa_minus` | Tvungen negativ (aggresjon/flukt) | 5 |

```
humanScore = round(WQ-spline(humanI, SPLINE.HUMAN_ANIMAL))
```

**Referanseverdier:**

| Resultat | humanI | humanScore |
|---|---|---|
| vaa_plus | 100 | 100 |
| faa_plus | 75 | 76 |
| faa_passive | 50 | 24 |
| vaa_minus | 25 | 13 |
| faa_minus | 5 | 3 |

**Brukerforklaring:** *"Tilnærmingstesten viser om hesten har et positivt, nøytralt eller negativt forhold til mennesker. Testen gjøres best av en ukjent person – eierens hest kan komme av vane, ikke av tillit."*

**Status:** ✓ HUMAN_ANIMAL-spline fra finsk protokoll.

### 4.2 Tester (`tester`) 📋

**Dagens status:** `S.tester` lagres (ukjent/kjent person), men brukes IKKE i beregningen. Skjemaet advarer om at eier-tester "kan gi unøyaktig resultat", men dette har ingen konsekvens.

**Foreslått beslutning:** Gi en modifikator på −10 humanScore-poeng ved `tester = 'known'`, klippet til min 0. Begrunnelse: testen er faglig mindre pålitelig med kjent person, og det bør reflekteres i score – ellers oppmuntrer skjemaet til å bruke eier som tester (mindre strevsomt, ingen straff). −10 er moderat nok til at det ikke ødelegger resultatet, men tydelig nok til å sende signal.

**Alternativ:** Vis en visuell advarsel i resultatet ("⚠ Test utført av kjent person – kan gi høyere score enn reell") uten score-konsekvens. Enklere, men bryter prinsippet om at observasjonen skal påvirke scoren.

---

## 5. Sub-indeks Idx4 – Emosjonell tilstand

Idx4 har **høyest enkelt-μ (0,17)** i K4-Choquet.

### 5.1 Registreringer

- `obsMinutes` 📋: Observasjonstid (samme som sosial observasjon)
- `frustration` 🐴: Antall hendelser (skraping, gjesping, hodekasting, vandring)
- `stereotype` 🐴: Antall hendelser (krybbebiting, luftsluking, veving)

### 5.2 Poengberegning

```
hours = obsMinutes / 60
stereoRate = stereotype / hours
frustRate = frustration / hours
emotionalScore = round(max(0, min(100, 100 − stereoRate · 10 − frustRate · 5)))
```

**Referanseverdier (ved 20 min observasjon):**

| Stereotypier | Frustrasjon | emotionalScore |
|---|---|---|
| 0 | 0 | 100 |
| 1 | 0 | 70 |
| 0 | 1 | 85 |
| 2 | 2 | 10 |
| 3 | 1 | 0 |

**Brukerforklaring:** *"Stereotypier (veving, krybbebiting) er tegn på at hestens behov ikke har vært møtt over tid – de veier dobbelt så tungt som frustrasjon (skraping, vandring). Én stereotypi i en 20-minutters observasjon reduserer scoren med 30 poeng, fordi det er et alvorlig signal."*

**Åpent spørsmål:** Koeffisientene 10 og 5 er ad-hoc. Finsk protokoll bruker prevalens-terskler. Beholder for nå fordi rate-basert scoring gir bedre oppløsning enn terskelbasert for N = 1, men verdiene bør valideres av Essi.

---

## 6. Aggregering til K4 – Choquet-integral

### 6.1 Håndtering av manglende sub-indekser

Minimum 3 av 4. Manglende fylles med gjennomsnitt. Dekningsgrad vises.

### 6.2 Eksempel

`socialScore = 70, pastureScore = 90, humanScore = 60, emotionalScore = 30`

Sortert: 30 (idx 4), 60 (idx 3), 70 (idx 1), 90 (idx 2)
- 30·μ₁₂₃₄ + 30·μ₁₂₃ + 10·μ₁₂ + 20·μ₂
- = 30·1,00 + 30·0,42 + 10·0,12 + 20·0,07 = 30 + 12,6 + 1,2 + 1,4 = **45**

**Brukerforklaring:** *"Atferdsscoren styres av det svakeste området. Stereotypier eller frustrasjon (Idx4) trekker hardest fordi de er direkte tegn på dårlig velferd. God beitetid alene kan ikke kompensere for stress-tegn."*

---

## 7. Oppsummeringstabell

| # | Registrering | Type | Sub-indeks | Maks pos | Maks neg |
|---|---|---|---|---|---|
| 1 | Scenario (0–3) | 🏠 | Idx1 | 100 | 5 |
| 2 | Indoor contact | 🏠 | Idx1 | +15 bonus | 0 |
| 3 | Sosial obs (pos/neg) | 🐴 | Idx1 | +20 mod | −20 mod |
| 4 | Beitedager | 📋 | Idx2 | 100 | 0 |
| 5 | Berikelsesbeiting | 📋 | Idx2 | Del av 100 | 0 |
| 6 | VAA/FAA-resultat | 🐴 | Idx3 | 100 | 3 |
| 7 | Tester (ukjent/kjent) | 📋 | Idx3 | 0 | −10 (foreslått) |
| 8 | Frustrasjon (telling) | 🐴 | Idx4 | 100 | → 0 |
| 9 | Stereotypier (telling) | 🐴 | Idx4 | 100 | → 0 (dobbel vekt) |

---

## 8. Gjenstående åpne spørsmål (runde 2/3)

| # | Spørsmål | Foreslått løsning | Status |
|---|---|---|---|
| 1 | tester-variabelen ikke brukt | −10 modifier ved kjent tester | ✓ Foreslått |
| 2 | μ₂ (beite) = 0,07 for lavt for norske forhold? | Beholder finsk verdi, flagg for Essi (runde 3) | 🔄 Essi |
| 3 | Koeffisienter 10/5 for stereo/frust | Beholder, validér med Essi | 🔄 Essi |
| 4 | negStrong · 3 i sosial obs | Beholder. 3× er forankret i at aggresjon er vesentlig alvorligere | ✓ Foreslått |
| 5 | Indoor-bonus +15 fast uavhengig av scenario | Beholder. Enkel og konsistent. | ✓ Foreslått |

---

## 9. Referanser i kode

| Element | Linjenummer (ca.) |
|---|---|
| `S`-state (K4-felter) | 1056–1060 |
| `selectScenario()` | 1273–1278 |
| `selectVAA()` | 1279–1284 |
| `SPLINE.PASTURE` | 1328–1333 |
| `SPLINE.HUMAN_ANIMAL` | 1334–1339 |
| `CHOQUET_CAPS.K4` | 1359–1366 |
| `calcK4()` | 1825–1915 |

HTML: `<div id="page-k4">` rundt linje 889–1005.
