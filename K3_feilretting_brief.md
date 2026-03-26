# Feilretting: Beregningsavvik mellom NHS-skjema og finsk referanseside

## Kontekst

`Skjema_hestevelferd_stall.html` er et digitalt vurderingsverktøy for hestevelferd basert på den finske Welfare Quality®-protokollen. Verktøyet beregner 12 delindekser (Idx1–Idx12) som aggregeres til 4 kategorier (K1–K4), som igjen gir en totalklassifisering.

Skjemaet har en F1-eksportfunksjon som genererer verdier til den finske nettsiden (https://hevostenhyvinvointi.fi/arviointi/). Når vi laster opp de eksporterte verdiene til den finske nettsiden, får vi **markant forskjellige resultater** fra hva NHS-skjemaet beregner internt. Den finske nettsiden er fasit ("ground truth").

Filen er én stor HTML-fil (~8100 linjer) med innebygd JavaScript. Den har to lag med beregningskode som ofte er i konflikt:
1. **WQ-funksjonsobjektet** (definert ca. linje 340–3570): Rene funksjonsdefinisjoner med spline-koeffisienter og Choquet-kapasiteter
2. **Inline beregningskode** (i updateK3Score(), ca. linje 6710–6970): Den faktiske koden som kjøres — denne avviker ofte fra WQ-funksjonene

Nøkkelprinsipp: **Beregningsfeil ligger nesten alltid i inline-koden, ikke i WQ-funksjonsdefinisjonene.** Sjekk alltid begge lag.

## Testdata (faktisk vurdering 5. mars 2026)

### Grunndata (A1)
- Totalt antall hester: 44
- Utvalg (otanta): 44
- K3 examinedCount (manuelt satt): 33

### K1-data
- BCS: kl1=0, kl2=2, kl3=31, kl4=11, kl5=0
- Vann: A3=67, A4=2 (gruppehold OK)
- Grovfôr: 2, Kraftfôr-rekkefølge: 0

### K2-data
- Liggekomfort: lying0=32, lying1=12, lying2=0 (total=44)
- Skitne hester: 1 av 44
- Termisk: 2 unike hester med funn av 44
- Uten ly: 3
- Bokstid: 0 under 14t, 44 mellom 14-18t, 0 over 18t

### K3-data (kliniske funn, N=33 undersøkt)
| Funn | Antall |
|------|--------|
| skinMild | 9 |
| skinSevere | 0 |
| swellingCold | 9 |
| swellingWarm | 4 |
| hoof | 0 |
| **scratch** | **16** |
| nasalClear | 0 |
| nasalThick | 1 |
| eyeClear | 0 |
| eyeThick | 0 |
| lameness | 0 |
| cough | 0 |
| diarrhoea | 3 |
| tack | 15 |
| back | 5 |
| mouthMild | 23 |
| mouthSevere | 19 |
| mouthNA | 0 |
| colic | 10 (siste 12 mnd, av 44 hester totalt) |
| saddlingN | 44, normal=19, mild=23, severe=2 |
| hgsN | 44, low=41, mild=3, severe=0 |

### K4-data
- totalHorses=44, scenario0=32, scenario2=12
- indoor=42, valintapuu=yes
- minutes=80, horsesObserved=30
- positive=2, negMild=9, negStrong=5
- frustration=2, stereotype=2
- pastureDays=80, pastureHorses=34
- vaaSampleSize=44, vaa_plus=35, vaa_minus=0
- faa_plus=6, faa_passive=3, faa_minus=0

## Score-sammenligning

### Delindekser (Idx1–Idx12)

| # | Delindeks | NHS | Finsk | Diff | Prioritet |
|---|-----------|-----|-------|------|-----------|
| 1 | BCS (Ei pitkittynyttä nälkää) | 86.9 | 74.0 | +12.9 | MEDIUM |
| 2 | Vann (Ei pitkittynyttä janoa) | 67 | 67.0 | **0** ✅ | — |
| 3 | Hvile+termisk (Lepo- ja lämpömukavuus) | 42.4 | 35.8 | +6.6 | LOW |
| 4 | Bevegelse (Liikkumismahdollisuudet) | 69 | 63.8 | +5.2 | LOW |
| 5 | Skader (Ei vauriota) | 78.9 | **35.4** | **+43.5** | **KRITISK** |
| 6 | Annen smerte (Ei muuta kipua) | 1.5 | **82.7** | **-81.2** | **KRITISK** |
| 7 | Sykdom (Ei sairauksia) | 24.7 | **50.2** | **-25.5** | **HØY** |
| 8 | Smerte fra bruk (Ei käytöstä aiheutuvaa kipua) | 1 | 11.7 | -10.7 | HØY |
| 9 | Sosial atferd | 81 | 85.3 | -4.3 | LOW |
| 10 | Andre atferdsmålere | 72 | 67.6 | +4.4 | LOW |
| 11 | Menneske-dyr | 85.8 | 83.8 | +2.0 | LOW |
| 12 | Positiv emosjonell | 36.4 | 38.1 | -1.7 | LOW |

### Kategorier (K1–K4)

| Kat | NHS | Finsk | Diff |
|-----|-----|-------|------|
| K1 | 69.4 | 67.8 | +1.6 |
| K2 | 54 | 46.2 | +7.8 |
| **K3** | **18** | **33.1** | **-15.1** |
| K4 | 53 | 52.6 | +0.4 |

## Identifiserte feil — prioritert rekkefølge

### FEIL 1 (KRITISK): Idx6/Idx8 — Feil gruppering av kliniske funn

**Symptom:** NHS Idx6=1.5 vs finsk 82.7, NHS Idx8=1 vs finsk 11.7

**Problem:** NHS-skjemaet plasserer `back` (rygg) og `tack` (trykkmerker) i **Idx6** (Annen smerte), men den finske protokollen plasserer dem i **Idx8** (Smerte fra bruk).

Nåværende Idx6 (linje ~6899-6907):
```javascript
// Feil: back og tack hører IKKE her
otherPainScore = Math.max(0, 100 - (backPct * 2) - (tackPct * 1.5));
// + HGS
```

Nåværende Idx8 (linje ~6873-6895):
```javascript
// Mangler: back og tack burde vært her sammen med munn og påsaling
mgmtScores = [mouthSpline, saddSpline]; // bare munn + påsaling
```

**Riktig gruppering (finsk protokoll):**
- **Idx6 (Ei muuta kipua):** HGS + kolikk
- **Idx8 (Ei käytöstä aiheutuvaa kipua):** Munn + påsaling + rygg + trykkmerker

**Merk:** Idx6 i den finske protokollen inkluderer sannsynligvis kolikk (som nå er i Idx7/disease). Sjekk dette mot `docs/How_the_welfare_assessment_calculation_system_works.docx`. Idx6 finsk=82.7 med data HGS mild=3, severe=0 (N=44) og colic=10 tyder på at kolikk gir liten innvirkning her, eller at den bruker en annen formel enn ren prosent.

### FEIL 2 (KRITISK): Idx5 — Scratch (gnagsår) mangler i integument-beregningen

**Symptom:** NHS Idx5=78.9 vs finsk 35.4

**Problem:** `scratch` (gnagsår, n=16 — det høyeste enkelttallet!) er IKKE inkludert i Idx5-beregningen. I inline-koden (linje ~6813-6815):
```javascript
const skinMildPct = (totals.skinMild + totals.swellingCold) / N;     // 9+9=18
const skinSeverePct = (totals.skinSevere + totals.swellingWarm) / N;  // 0+4=4
const lamenessPct = (totals.lameness + totals.hoof) / N;              // 0+0=0
// scratch (16 stk.) er IKKE med noe sted!
```

**Mulig fix:** Scratch bør sannsynligvis legges til skinMild i integument-beregningen:
```javascript
const skinMildPct = (totals.skinMild + totals.swellingCold + totals.scratch) / N;
```
Alternativt kan scratch ha sin egen vekting. Verifiser ved å teste: med scratch=16 inkludert som mild i integument, hva gir Idx5? Sammenlign med finsk 35.4.

### FEIL 3 (HØY): Idx7 — Sykdomsberegning med feil spline og feil gruppering

**Symptom:** NHS Idx7=24.7 vs finsk 50.2

**Problem A: Spline-mismatch.** Inline-koden (linje 6865) bruker knot=65 med koeffisienter:
```javascript
// Inline bruker DISSE (knot=65):
low: 0.5280510652, -0.0036474543, 0.0000595889
high: -154.2417024020, 7.6468988725, -0.1131681899, 0.0006212337
```
Men `WQ.SPLINE.DISEASE` (linje 3248) har knot=70 med helt andre koeffisienter:
```javascript
// WQ-objektet har DISSE (knot=70):
low: { a: 0, b: 0.39094656, c: 0.00217984, d: 0.000030794 }
high: { a: -105.607674, b: 4.91698974, c: -0.06247792, d: 0.00033869 }
```
Vet ikke hvilken som er riktig. Test begge mot finsk resultat.

**Problem B: Area-gruppering.** Inline-koden bruker 4 grupperte areas (ORL, respiratory, diarrhoea, colic). `WQ.calcDisease()` (linje ~3488) bruker 5 individuelle areas. Finsk side bruker trolig 4 areas (som WQ cattle protocol). Sjekk om kolikk faktisk hører til Idx7 eller Idx6 (se Feil 1).

**Problem C: Nevner.** Inline bruker N=33 (examinedCount). Finsk side har bare tilgang til A1-data der total=44 og sample=44.

### FEIL 4 (HØY): K3-aggregering bruker vektet gjennomsnitt i stedet for Choquet

**Symptom:** K3-kategoriscoren er feil uavhengig av delindeksene

**Problem:** Inline-koden (linje 6912-6921) bruker:
```javascript
indexScores.sort((a, b) => a - b);
const weights = [0.35, 0.30, 0.20, 0.15];
```
Men `WQ.calcK3()` (linje 3526-3529) bruker Choquet-integral med K3-kapasiteter:
```javascript
calcK3: function(injuryScore, otherPainScore, diseaseScore, mgmtPainScore) {
    const scores = [injuryScore, otherPainScore, diseaseScore, mgmtPainScore];
    return this.choquet(scores, this.CHOQUET.K3);
}
```
Choquet K3-kapasitetene (linje 3294-3302) ser riktige ut (hentet fra finsk protokoll Taulukko 18).

**Fix:** Erstatt det vektede gjennomsnittet med et kall til `WQ.calcK3(injuryScore, otherPainScore, diseaseScore, painMgmtScore)`.

### FEIL 5 (MEDIUM): Idx1 — BCS-avvik

**Symptom:** NHS Idx1=86.9 vs finsk 74.0

**Problem:** Med data kl=[0,2,31,11,0] N=44:
- severitySum = 0×1.0 + 2×0.25 + 0×0.5 = 0.5
- R = 100 - (0.5/44)×100 = 98.86
- spline(98.86, knot=80) → high segment → NHS gir 86.9

Finsk gir 74.0 — stor forskjell. Mulige årsaker:
1. Feil BCS-vekter (er kl4 virkelig 0? Noen protokoller gir kl4 en liten vekt)
2. Feil spline-koeffisienter
3. Annen formel for R (kanskje % utenfor optimal range?)

Sjekk `docs/How_the_welfare_assessment_calculation_system_works.docx` for detaljer om BCS-formelen.

### FEIL 6 (LOW): K2-avvik og Idx3/Idx4

NHS K2=54 vs finsk 46.2, Idx3 42.4 vs 35.8, Idx4 69 vs 63.8. Bør undersøkes etter K3 er fikset.

## Arbeidsmetode

1. **Gjør én endring om gangen** — commit etter hver logisk fix
2. **Test mot de finske referansetallene** etter hver endring — beregn forventet score manuelt
3. **Sjekk alltid begge lag** (WQ-funksjoner vs inline-kode) før du endrer noe
4. **Bruk `grep -n`** med flere søkeord for å finne alle relaterte kodeseksjoner
5. **Ikke endre F1-eksporttabellen** — feltrekkefølgen og finske feltnavn er korrekte
6. **Ikke endre WQ.choquet()** eller WQ.spline() med mindre du har bevis for at de er feil
7. **Les referansedokumentene** i `docs/` — spesielt `How_the_welfare_assessment_calculation_system_works.docx`

## Referansefiler i repo

- `docs/How_the_welfare_assessment_calculation_system_works.docx` — Forklaring av beregningssystemet
- `docs/Håndbok_hestevelferdsindikatorer_oversatt.docx` — Oversatt håndbok med indikatorbeskrivelser

## Nevner-logikk (viktig kontekst)

Den finske nettsiden kjenner bare til verdiene i F1-eksporten:
- A1: total=44, sample=44
- A8: kliniske funn (rå tall, f.eks. skinMild=9)

Den beregner rater som `skinMild / sample`. NHS-skjemaet har en K3-spesifikk examinedCount=33 som ikke eksporteres. Når vi sammenligner med finsk side, bruker finsk side 44 som nevner. Enten må vi eksportere examinedCount, eller akseptere at sammenligningen bruker N=44.

For denne feilrettingen: bruk N=44 (finsk sides perspektiv) når du beregner forventede scorer for sammenligning.
