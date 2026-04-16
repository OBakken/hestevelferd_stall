# Claude Code Brief: Implementering av revidert scoring – hestevelferd_en_hest.html

**Dato:** 2026-04-16
**Mål:** Oppdatere `hestevelferd_en_hest.html` basert på alle vedtatte revisjonsbeslutninger dokumentert i `/docs/formler/formelbeskrivelse_oversikt_enhest.md` og tilhørende K1–K4-dokumenter.
**Commit-melding:** `Implementerer revidert scoring: gradering, severity-splines, fill-with-avg, WQ-klassifisering`

---

## Overordnet kontekst

Én-hest-versjonen av hestevelferdsskjemaet (~2330 linjer, én HTML-fil med inline JS) revideres med 7 prinsippbeslutninger som påvirker beregningslogikk, UI og resultatvisning. Alle formler er dokumentert i 5 MD-filer som skal kopieres til `/docs/formler/` i repoet.

**Viktig:** Denne filen ER hele spesifikasjonen. Ikke gjett – hvis noe er uklart, les de refererte MD-filene.

---

## DEL 1: Kopier MD-filer til repo

Kopier disse 5 filene til `/docs/formler/`:
- `formelbeskrivelse_oversikt_enhest.md`
- `formelbeskrivelse_K1_ernaering_enhest.md`
- `formelbeskrivelse_K2_oppstalling_enhest.md`
- `formelbeskrivelse_K3_helse_enhest.md`
- `formelbeskrivelse_K4_atferd_enhest.md`

---

## DEL 2: HTML – Gradér alle binære målinger til 0/1/2

Erstatt alle binære `radio-group` (Nei/Ja) med tre-valg (0/1/2) for følgende registreringer. Behold eksisterende CSS-klasser (`score-radio green/yellow/red` for 0/1/2).

### K3 – Helse (de fleste endringene)

**Halthet (`lameness`)** – erstatt Nei/Ja med:
- 0: Ingen synlig halthet
- 1: Mild (kun synlig i trav, ikke i skritt)
- 2: Alvorlig (synlig i skritt, tydelig nikking/vipping)

**Ryggsmerte (`back`)** – erstatt Nei/Ja med:
- 0: Ingen reaksjon
- 1: Mild respons (reagerer ett punkt, viker unna)
- 2: Kraftig respons (flere steder, snapper, slår)

**Hovproblemer (`hoof`)** – erstatt Nei/Ja med:
- 0: Ingen hovproblemer
- 1: Mild (én hov, lette sprekker)
- 2: Alvorlig (flere hover, byll, tydelig overgrodd)

**Gnagemerker (`scratch`)** – erstatt Nei/Ja med:
- 0: Ingen gnagemerker
- 1: Lett (noe tynnslitt hår)
- 2: Tydelig (avgnagd man, bare flekker)

**Hoste (`cough`)** – erstatt Nei/Ja med:
- 0: Ingen hoste observert
- 1: Sporadisk (enkeltstående hoste)
- 2: Gjentakende (flere hosteanfall)

**Diaré (`diarrhoea`)** – erstatt Nei/Ja med:
- 0: Normal avføring
- 1: Løs avføring (men ikke tilsølt)
- 2: Tilsølt bakpart

**Kolikk (`colic`)** – erstatt Nei/Ja med:
- 0: Ingen kolikk siste 12 måneder
- 1: Én episode (ikke veterinærbehandlet)
- 2: Behandlet av veterinær eller gjentakende

**Utstyrsskader (`tack`)** – erstatt Nei/Ja med:
- 0: Ingen utstyrsskader
- 1: Mild (gnag uten åpent sår)
- 2: Alvorlig (åpent sår fra utstyr)

### K2 – Oppstalling

**Temperaturstress (`thermal`)** – erstatt fire spesifikke verdier (ok/shiver/sweat/pant) med:
- 0: Ingen tegn observert
- 1: Milde tegn (litt urolig, noe svette uten anstrengelse)
- 2: Tydelig stress (skjelving, kraftig svetting, anstrengt pust)

**Tilgang til ly (`shelter`)** – erstatt Ja/Nei med:
- 0: Godkjent ly (leskur med tak + 3 vegger, eller tett skog)
- 1: Delvis ly (takutspring, glissen skog, vindskjerm uten tak)
- 2: Ingen ly

**Renhet (`clean`)** – erstatt Ja/Nei med:
- 0: Ren (ingen fastsittende skitt)
- 1: Lett tilsmusset (noe skitt, men ikke fastsittende)
- 2: Fastsittende skitt (inntørket, > kredittkort)

### K1 – Ernæring

**Vann ute (`waterOut`)** – legg til et tredje valg:
- `na`: Ikke aktuelt (hesten er aldri ute)
Håndteres som `waterIn = na` – utelates fra C2.

---

## DEL 3: HTML – Målingstype-ikoner

Legg til et lite ikon foran hver `section-title` eller `form-label` som viser målingstype:

- 🐴 = Dyrbasert (observasjon av hesten)
- 🏠 = Ressursbasert (tilstedeværelse av ressurs)
- 📋 = Styringsbasert (rutine/intervju)

**K1:** BCS → 🐴, feedInterval → 📋, feedOrder → 📋, feedAmount → 📋, waterIn → 🏠, waterOut → 🏠, waterFlow → 🏠, waterPoints → 🏠
**K2:** boxTime → 📋, scan → 🐴, resources → 🏠, enrichment → 🏠, bedding → 🏠, clean → 🐴, thermal → 🐴, shelter → 🏠
**K3:** Alle 🐴 unntatt colic → 📋
**K4:** scenario → 🏠, indoorContact → 🏠, sosial obs → 🐴, pastureDays → 📋, enrichDays/Hours → 📋, tester → 📋, vaaResult → 🐴, frustration → 🐴, stereotype → 🐴

Implementer som et lite `<span>` med klassen `measure-type` foran teksten. CSS: `font-size: 0.85em; margin-right: 0.3rem;`

---

## DEL 4: JS State – Oppdater S-objekt

Oppdater `const S = {...}` for å reflektere graderte verdier. Felter som endres fra string til int:
```javascript
// K2 - nye graderte (var binær string, nå int 0/1/2)
thermal: null,   // var string 'ok'/'shiver'/etc, nå int 0/1/2
shelter: null,   // var string 'yes'/'no', nå int 0/1/2
clean: null,     // var string 'yes'/'no', nå int 0/1/2

// K3 - nye graderte (var string 'yes'/'no', nå int 0/1/2)
hoof: null, scratch: null, lameness: null,
cough: null, diarrhoea: null, colic: null, tack: null,
back: null,
```

Oppdater alle tilhørende `document.addEventListener('change', ...)` handlers til å parse `parseInt(val)` for disse feltene.

---

## DEL 5: JS Beregning – Nye hjelpefunksjoner

### 5.1 Severity-til-spline-score

```javascript
// Beregner score fra severity-gradering via WQ-spline
// mildWeight/severeWeight: finske vekter (0.2/1.0 for integument, 0.5/1.0 for andre)
function severityToSplineScore(grade, mildWeight, severeWeight, splineConfig) {
    if (grade === null || grade === undefined) return null;
    const g = parseInt(grade);
    if (g === 0) return Math.round(wqSpline(100, splineConfig));
    const virtualPrev = g === 1 ? mildWeight : severeWeight;
    const I = Math.max(0, 100 - virtualPrev * 100);
    return Math.round(wqSpline(I, splineConfig));
}
```

### 5.2 Dekningsgrad-visning

```javascript
// Viser dekningsgrad i score-overskrift
function formatScore(score, registered, total) {
    if (score === null) return '–';
    return `${score} (${registered}/${total})`;
}
```

### 5.3 WQ-klassifisering

```javascript
function wqClassify(k1, k2, k3, k4) {
    const scores = [k1, k2, k3, k4].filter(s => s !== null);
    if (scores.length < 4) return null;

    const all = [k1, k2, k3, k4];
    const sorted = [...all].sort((a, b) => b - a);

    // Excellent: >55 alle, >80 på minst to
    if (all.every(s => s > 55) && sorted.filter(s => s > 80).length >= 2)
        return { level: 'Excellent', color: '#16a34a', emoji: '⭐' };

    // Enhanced: >20 alle, >55 på minst to
    if (all.every(s => s > 20) && sorted.filter(s => s > 55).length >= 2)
        return { level: 'Enhanced', color: '#2563eb', emoji: '✅' };

    // Acceptable: >10 alle, >20 på minst tre
    if (all.every(s => s > 10) && sorted.filter(s => s > 20).length >= 3)
        return { level: 'Acceptable', color: '#d97706', emoji: '⚠️' };

    return { level: 'Not classified', color: '#dc2626', emoji: '🔴' };
}
```

Vis WQ-klassifiseringen på resultatsiden (`page-result`) under de fire K-barene. Legg til et `<div id="wqClassification">` med nivå, emoji og forklarende tekst.

---

## DEL 6: JS Beregning – Endringer per kategori

### 6.1 calcK1()

**Endre C2 worst-dominance fra 0,65 til 0,70:**
```javascript
// GAMMEL:
waterScore = Math.round(waterComponents[0] * 0.65 + (...) * 0.35);
// NY:
waterScore = Math.round(waterComponents[0] * 0.70 + (...) * 0.30);
```

**Legg til `waterOut = 'na'` håndtering:**
```javascript
if (S.waterOut === 'na') {
    // Utelat fra waterComponents, som waterIn = na
}
```

**Refaktorér K1 Choquet til å bruke wqChoquet():**
```javascript
// GAMMEL: håndkodet mn/mx/muSingle
// NY:
const k1indices = [
    {score: c1Score, idx: 1},
    {score: waterScore, idx: 2}
];
return wqChoquet(k1indices, CHOQUET_CAPS.K1);
```

---

### 6.2 calcK2()

**Flytt scan fra Idx2 til Idx1:**

I Idx1-seksjonen, etter `omaehtoinenScore` er beregnet fra resources + enrichment:
```javascript
// Scan modifiserer omaehtoinenScore (flyttet fra Idx2)
if (scanScore !== null && omaehtoinenScore !== null) {
    omaehtoinenScore = Math.round(omaehtoinenScore * 0.6 + scanScore * 0.4);
} else if (scanScore !== null && omaehtoinenScore === null) {
    omaehtoinenScore = scanScore;
}
```

Fjern all scan-logikk fra Idx2 (liggekomfort). `lyingScore = beddingSplineScore` (direkte, uten scan-cap).

**Graderte verdier for thermal:**
```javascript
// GAMMEL: thermalStress = S.thermal === 'ok' ? 100 : 0;
// NY:
if (S.thermal !== null) {
    thermalStress = {0: 100, 1: 50, 2: 0}[parseInt(S.thermal)] ?? null;
}
```

**Graderte verdier for shelter:**
```javascript
// GAMMEL: shelterScore = S.shelter === 'yes' ? 100 : 0;
// NY:
if (S.shelter !== null) {
    shelterScore = {0: 100, 1: 50, 2: 0}[parseInt(S.shelter)] ?? null;
}
```

**Graderte verdier for clean:**
```javascript
// GAMMEL: cleanScore = S.clean === 'yes' ? 100 : 30;
// NY:
if (S.clean !== null) {
    cleanScore = {0: 100, 1: 70, 2: 30}[parseInt(S.clean)] ?? null;
}
```

**Fill missing → gjennomsnitt (erstatter "fill with best"):**
```javascript
// GAMMEL:
const best = Math.max(...availableK2);
if (movementScore === null) movementScore = best;
// NY:
const avg = Math.round(availableK2.reduce((a,b) => a+b, 0) / availableK2.length);
if (movementScore === null) movementScore = avg;
if (comfortScore === null) comfortScore = avg;
if (thermalScore === null) thermalScore = avg;
```

**Minimum-krav (2 av 3):**
```javascript
if (availableK2.length < 2) return null;
```

---

### 6.3 calcK3()

**Integument-gruppe: bruk SKIN-spline med severity 0,2/1,0:**
```javascript
// GAMMEL: integParts.push({0:100, 1:65, 2:10}[parseInt(S.skin)] ?? null);
// NY:
if (S.skin !== null && S.skin !== undefined) {
    integParts.push(severityToSplineScore(S.skin, 0.2, 1.0, SPLINE.SKIN));
}
if (S.swelling !== null && S.swelling !== undefined) {
    integParts.push(severityToSplineScore(S.swelling, 0.2, 1.0, SPLINE.SKIN));
}
if (S.hoof !== null) {
    integParts.push(severityToSplineScore(S.hoof, 0.2, 1.0, SPLINE.SKIN));
}
if (S.scratch !== null) {
    integParts.push(severityToSplineScore(S.scratch, 0.2, 1.0, SPLINE.SKIN));
}
```

**Halthet: direkte mapping (LAMENESS-spline problematisk for N=1):**
```javascript
// GAMMEL: lameScore = S.lameness === 'no' ? 100 : 0;
// NY:
if (S.lameness !== null) {
    lameScore = {0: 100, 1: 50, 2: 0}[parseInt(S.lameness)] ?? null;
}
```

**Ryggsmerte: direkte mapping:**
```javascript
// GAMMEL: otherParts.push(S.back === 'no' ? 100 : 0);
// NY:
if (S.back !== null) {
    otherParts.push({0: 100, 1: 50, 2: 0}[parseInt(S.back)] ?? null);
}
```

**Sykdom: graderte alarm-verdier + maxAlarms 7,5:**
```javascript
// Erstatt all sykdoms-alarm-logikk med:
if (S.nasal !== null && S.nasal !== undefined) {
    diseaseChecked++;
    const g = parseInt(S.nasal);
    if (g === 1) diseaseAlarms += 0.5;
    else if (g === 2) diseaseAlarms += 1.0;
}
if (S.eye !== null && S.eye !== undefined) {
    diseaseChecked++;
    const g = parseInt(S.eye);
    if (g === 1) diseaseAlarms += 0.5;
    else if (g === 2) diseaseAlarms += 1.0;
}
if (S.cough !== null) {
    diseaseChecked++;
    const g = parseInt(S.cough);
    if (g === 1) diseaseAlarms += 0.5;
    else if (g === 2) diseaseAlarms += 1.0;
}
if (S.diarrhoea !== null) {
    diseaseChecked++;
    const g = parseInt(S.diarrhoea);
    if (g === 1) diseaseAlarms += 0.75;
    else if (g === 2) diseaseAlarms += 1.5;
}
if (S.colic !== null) {
    diseaseChecked++;
    const g = parseInt(S.colic);
    if (g === 1) diseaseAlarms += 1.5;
    else if (g === 2) diseaseAlarms += 3.0;
}

if (diseaseChecked > 0) {
    const maxAlarms = 7.5;  // ENDRET fra 7
    const diseaseI = Math.max(0, 100 - (diseaseAlarms / maxAlarms) * 100);
    diseaseScore = Math.round(wqSpline(diseaseI, SPLINE.DISEASE));
}
```

**Utstyrsskader: gradert:**
```javascript
// GAMMEL: mgmtParts.push({score: S.tack === 'no' ? 100 : 20, weight: 1.0});
// NY:
if (S.tack !== null) {
    mgmtParts.push({0: 100, 1: 60, 2: 10}[parseInt(S.tack)] ?? null);
}
```

**mgmtPainScore: forenkle til weighted min-max (0,70):**
```javascript
// GAMMEL: kompleks vektet formel med intern weight
// NY:
const mgmtValid = [mouthScore, tackScore, saddScore].filter(s => s !== null);
if (mgmtValid.length > 0) {
    mgmtValid.sort((a, b) => a - b);
    if (mgmtValid.length === 1) {
        mgmtPainScore = Math.round(mgmtValid[0]);
    } else {
        const restAvg = mgmtValid.slice(1).reduce((a,b) => a+b, 0) / (mgmtValid.length - 1);
        mgmtPainScore = Math.round(mgmtValid[0] * 0.70 + restAvg * 0.30);
    }
}
```

Der `mouthScore`, `tackScore`, `saddScore` beregnes som direkte mappinger:
```javascript
let mouthScore = null;
if (S.mouth !== null && S.mouth !== undefined && S.mouth !== 'na') {
    mouthScore = {0: 100, 1: 40, 2: 5}[parseInt(S.mouth)] ?? null;
}
let tackScore = null;
if (S.tack !== null) {
    tackScore = {0: 100, 1: 60, 2: 10}[parseInt(S.tack)] ?? null;
}
let saddScore = null;
if (S.saddlingRelevant !== 'no' && S.saddlingScore !== null) {
    saddScore = {0: 100, 1: 45, 2: 5}[parseInt(S.saddlingScore)] ?? null;
}
```

**Fill missing → gjennomsnitt + minimum 3/4:**
```javascript
const k3scores = [injuryScore, otherPainScore, diseaseScore, mgmtPainScore];
const availableK3 = k3scores.filter(s => s !== null);
if (availableK3.length < 3) return null;  // Minimum 3 av 4
if (availableK3.length < 4) {
    const avg = Math.round(availableK3.reduce((a,b) => a+b, 0) / availableK3.length);
    if (injuryScore === null) injuryScore = avg;
    if (otherPainScore === null) otherPainScore = avg;
    if (diseaseScore === null) diseaseScore = avg;
    if (mgmtPainScore === null) mgmtPainScore = avg;
}
```

---

### 6.4 calcK4()

**Tester-modifier:**
```javascript
// Etter humanScore er beregnet fra HUMAN_ANIMAL-spline:
if (humanScore !== null && S.tester === 'known') {
    humanScore = Math.max(0, humanScore - 10);
}
```

**Fill missing → gjennomsnitt + minimum 3/4:**
```javascript
const k4scores = [socialScore, pastureScore, humanScore, emotionalScore];
const availableK4 = k4scores.filter(s => s !== null);
if (availableK4.length < 3) return null;
if (availableK4.length < 4) {
    const avg = Math.round(availableK4.reduce((a,b) => a+b, 0) / availableK4.length);
    if (socialScore === null) socialScore = avg;
    if (pastureScore === null) pastureScore = avg;
    if (humanScore === null) humanScore = avg;
    if (emotionalScore === null) emotionalScore = avg;
}
```

---

## DEL 7: UI – Dekningsgrad og WQ-klassifisering

### 7.1 Dekningsgrad i recalc-funksjoner

Oppdater `recalcK1()` – `recalcK4()` til å spore dekningsgrad og vise den:

```javascript
function recalcK1() {
    // Tell registrerte sub-indekser
    const c1Ready = (S.bcsClass !== null) || (S.feedInterval !== null || S.feedOrder !== null || S.feedAmount !== null);
    const c2Ready = waterComponents.length > 0; // fra calcK1-logikk
    const registered = (c1Ready ? 1 : 0) + (c2Ready ? 1 : 0);
    const v = calcK1();
    updateProgress('K1', v);
    document.getElementById('k1Score').textContent = v !== null ? `${v} (${registered}/2)` : '–';
}
// Tilsvarende for K2 (x/3), K3 (x/4), K4 (x/4)
```

### 7.2 WQ-klassifisering på resultatsiden

Legg til i `updateResults()`:
```javascript
function updateResults() {
    // ... eksisterende K1-K4 barvisning ...

    // WQ-klassifisering
    const k1v = calcK1(), k2v = calcK2(), k3v = calcK3(), k4v = calcK4();
    const wq = wqClassify(k1v, k2v, k3v, k4v);
    const wqEl = document.getElementById('wqClassification');
    if (wq) {
        wqEl.innerHTML = `<div style="text-align:center;padding:1.5rem;">
            <div style="font-size:2rem;">${wq.emoji}</div>
            <div style="font-size:1.4rem;font-weight:800;color:${wq.color};">${wq.level}</div>
            <div style="font-size:0.85rem;color:var(--text-muted);margin-top:0.5rem;">
                Welfare Quality® klassifisering basert på K1–K4
            </div>
        </div>`;
    } else {
        wqEl.innerHTML = '<p style="text-align:center;color:var(--text-muted);">Fyll ut alle kategorier for WQ-klassifisering</p>';
    }
}
```

Legg til `<div id="wqClassification"></div>` i HTML-resultatdelen, etter de fire K-barene.

### 7.3 Rapport-tekst

Legg til i `drawShareCard()` eller i resultatvisningen:
```
"Denne vurderingen er basert på observasjon av én hest på ett tidspunkt.
Score-verdiene bruker Welfare Quality®-rammeverket med severity-vektet
virtuell prevalens for N = 1, og kan ikke direkte sammenlignes med
gruppe-/stallvurderinger."
```

---

## DEL 8: Testkriterier

Kjør følgende manuelle tester etter implementering:

| Test | Forventet resultat |
|---|---|
| BCS alle klasse 3, alt annet optimalt | K1 = 100, K2 = 100, K3 = 100, K4 = 100, Excellent |
| BCS klasse 1, resten optimalt | K1 < 20 (BCS = 0 → C1 = 0 → K1 dominert av C1) |
| Kun K1 og K2 fylt, K3/K4 tomme | K3 = "–", K4 = "–", WQ = "Fyll ut alle..." |
| K3: kun 2 av 4 sub-indekser | K3 = "–" (under minimum 3/4) |
| K3: 3 av 4 sub-indekser | K3 vises med "(3/4)", manglende = gjennomsnitt |
| skin=1 (mild) | integScore fra SKIN-spline(80) ≈ 57 (ikke 65 som før) |
| lameness=1 (mild) | lameScore = 50 (direkte mapping, ikke spline) |
| lameness=2 (alvorlig) | lameScore = 0 |
| thermal=1 (mild) | thermalStress = 50 (ikke 0 som før) |
| shelter=1 (delvis) | shelterScore = 50 (ikke 0 som før) |
| vaaResult = vaa_plus, tester = known | humanScore = 100 − 10 = 90 (tester-modifier) |
| vaaResult = vaa_plus, tester = unknown | humanScore = 100 (ingen modifier) |
| C2 vann: worst = 20 | Bruker 0,70 vekting (ikke 0,65) |
| Alle K1-K4 > 55, to > 80 | WQ = "Excellent" |
| K1=60, K2=15, K3=60, K4=60 | WQ = "Not classified" (K2 < 20) |

---

## DEL 9: Rekkefølge

1. Kopier MD-filer til `/docs/formler/`
2. Legg til `severityToSplineScore()`, `formatScore()`, `wqClassify()`
3. Oppdater `S`-state og event handlers for graderte verdier
4. Oppdater HTML for graderte radioknapper + målingstype-ikoner + waterOut na + wqClassification div
5. Oppdater `calcK1()` (C2 vekting 0,70 + waterOut na + refaktorér til wqChoquet)
6. Oppdater `calcK2()` (flytt scan, gradér thermal/shelter/clean, fill-with-avg)
7. Oppdater `calcK3()` (SKIN-spline, direkte lameness/back, graderte alarmer, maxAlarms 7,5, forenklet mgmt, fill-with-avg)
8. Oppdater `calcK4()` (tester-modifier, fill-with-avg)
9. Oppdater `recalcK1-K4()` for dekningsgrad
10. Oppdater `updateResults()` for WQ-klassifisering + rapport-tekst
11. Kjør testkriteriene i DEL 8
12. Commit
