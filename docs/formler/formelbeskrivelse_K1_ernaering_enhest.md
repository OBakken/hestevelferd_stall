# Formelbeskrivelse K1 – God ernæring (en-hest-versjon)

**Gjelder:** `hestevelferd_en_hest.html` (én-hest-versjonen)
**Kategori:** K1 – God ernæring
**Sub-indekser:** C1 (fravær av langvarig sult) og C2 (fravær av langvarig tørste)
**Sist oppdatert:** 2026-04-16 (runde 1 beslutninger innarbeidet)
**Status:** ✓ Prinsippspørsmål avgjort. Kategori-intern revisjon gjenstår for punkter merket 🔄.

---

## 1. Oversikt over K1

K1 kombinerer to sub-indekser til én samlet ernæringsscore (0–100):

| Sub-indeks | Hva måles | Registreringer | Målingstyper |
|---|---|---|---|
| **C1** – Fravær av langvarig sult | Hestens hold (utfall) pluss fôrrutiner (risiko) | a) BCS 🐴, b) fôrintervall 📋, c) fôringsrekkefølge 📋, d) kraftfôrmengde 📋 |  1 dyrbasert + 3 styringsbasert |
| **C2** – Fravær av langvarig tørste | Tilgang på rent vann og god flyt | e) vann inne 🏠, f) vann ute 🏠, g) vannstrøm 🏠, h) vannpunkter 🏠 | 4 ressursbasert |

**Målingstype-ikoner:** 🐴 = dyrbasert (observasjon av hesten), 🏠 = ressursbasert (tilstedeværelse av ressurs), 📋 = styringsbasert (rutine, intervju). Se oversiktsdokument seksjon 4.1 for begrunnelse.

**Sammensetning:** K1 beregnes med en to-variabel Choquet-integral (se seksjon 4) med kapasitetene:
- μ₁ (BCS-komponent C1 alene) = **0,12**
- μ₂ (vann-komponent C2 alene) = **0,27**
- μ₁₂ (begge sammen) = **1,00**

Den svakeste komponenten dominerer totalscoren. Vann har større individuell vekt enn hold, noe som reflekterer at akutt vannmangel er mer presserende enn gradvis holdendring. Kapasitetene følger finsk protokoll.

**Dekningsgrad:** K1 beregnes og vises hvis minst 1 av 2 sub-indekser er registrert. Dekningsgrad vises alltid: f.eks. `K1: 72 (2 av 2)`. Se oversiktsdokument seksjon 3 for beslutning om minimum-krav.

---

## 2. Sub-indeks C1 – Fravær av langvarig sult

C1 består av én utfallsmåling (BCS 🐴) og tre risiko-/rutinemålinger (fôring 📋). Logikken er:

```
C1 = max(0, min(100, bcsScore − feedPenalty))
```

Hvis BCS mangler, men fôrdata finnes: `C1 = max(0, 100 − feedPenalty)`.
Hvis ingenting er registrert: C1 = null (blir ikke brukt i Choquet).

### 2.1 Registrering a – Holdvurdering (BCS) 🐴

**Hva registreres:** Assessor vurderer hesten på 6 kroppsdeler. Hver kroppsdel får en klasse 1–5:

| Kroppsdel | Variabel |
|---|---|
| Hals/mankekam | `bcsA` |
| Sider/ribbein | `bcsB` |
| Manke/rygg | `bcsC` |
| Skulder | `bcsD` |
| Hofte/bekken | `bcsE` |
| Bakpart/haleansats | `bcsF` |

**Klasseskala:**
1 = svært tynn (kritisk), 2 = tynn, 3 = normalt hold (ideelt), 4 = overvektig, 5 = svært overvektig (kritisk)

**Aggregering til én BCS-klasse:** Flertallsavstemning – den klassen som flest kroppsdeler havner i, blir hestens BCS-klasse. Ved uavgjort brukes medianen. Krever minst 3 registrerte kroppsdeler.

**Poengberegning:**

Denne målingen følger allerede severity-vektet virtuell prevalens-logikken (se oversiktsdokument seksjon 4.3). Severity-mapping:

| BCS-klasse | Severity | R = 100 − severity·100 |
|---|---|---|
| 1 (svært tynn) | 1,00 | 0 |
| 2 (tynn) | 0,25 | 75 |
| 3 (normalt) | 0,00 | 100 |
| 4 (overvektig) | 0,00 | 100 |
| 5 (svært overvektig) | 0,50 | 50 |

**bcsScore** beregnes med WQ BCS-splinen (knot ved R = 80):
- For R ≤ 80: `score = 0,2217·R − 0,00277·R² + 0,0000593·R³`
- For R > 80: `score = −2961,32 + 111,27·R − 1,3909·R² + 0,00584·R³`

**Referanseverdier:**

| BCS-klasse | R | bcsScore (ca.) |
|---|---|---|
| 1 | 0 | 0 |
| 2 | 75 | 66 |
| 3 / 4 | 100 | 100 |
| 5 | 50 | 37 |

**Foreslått beslutning – BCS klasse 4 (overvektig):** Beholder severity 0 (= full score). Begrunnelse: i WQ-rammeverket er moderat overvekt en risikofaktor, ikke et akutt velferdsproblem i seg selv. Hesten med BCS 4 kan ha utmerket velferd ellers. Overvektsrisiko fanges bedre opp av fôrrutine-spørsmålene (c, d) som styringsindikatorer. Eventuelt kan dette revurderes hvis finsk protokoll har en annen tilnærming.

**Brukerforklaring:** *"BCS måler hestens faktiske hold – det er sluttfacit for ernæring. Klasse 3 er ideelt og gir full score. Svært tynn (1) eller svært overvektig (5) gir lav score fordi begge er helserisiko. Moderat overvekt (4) gir ikke trekk i seg selv, men dårlige fôrrutiner vil dra scoren ned via andre spørsmål."*

**Status:** ✓ Verifisert mot finsk kode.

---

### 2.2 Registrering b – Fôrintervall (`feedInterval`) 📋

**Hva registreres:** Lengste tidsrom hesten står uten tilgang på grovfôr, inkludert natt. Tall i timer.

**Poengberegning (straffeverdi):**

| Intervall (timer) | Straff |
|---|---|
| ≤ 4 | 0 |
| > 4 og ≤ 6 | 6 |
| > 6 og ≤ 8 | 15 |
| > 8 | `min(100, round(15 + 85·((h − 8)/16)^1,3))` |

Eksempler over 8 timer: 10 t ≈ 25, 12 t ≈ 37, 14 t ≈ 50, 16 t ≈ 66, 20 t ≈ 92, 24 t = 100.

**Brukerforklaring:** *"Hester produserer magesyre hele tiden og bør ikke stå mer enn 4 timer uten grovfôr. Jo lenger intervallet er, jo mer trekker det. Over 8 timer akselererer straffen fordi risikoen for magesår og stereotypier øker raskt."*

**Status:** ✓ Beholdes. Kalibreringen er spesifikk for én-hest-versjonen (stall-versjonen har en annen, mildere kurve). For én hest er det riktig at fôrrutinen veier mer direkte, fordi det ikke er et flokk-gjennomsnitt som demper effekten.

---

### 2.3 Registrering c – Fôringsrekkefølge (`feedOrder`) 📋

**Hva registreres:** Rekkefølge på grovfôr og kraftfôr.

| Verdi | Beskrivelse | Straff |
|---|---|---|
| `hay_first` | Grovfôr først | 0 |
| `no_concentrate` | Kun grovfôr | 0 |
| `same_time` | Samtidig | 10 |
| `concentrate_first` | Kraftfôr først | 25 |

**Brukerforklaring:** *"Kraftfôr på tom mage gir rask pH-endring i magen – det er en risikofaktor for fordøyelsesproblemer. Grovfôr først er optimal praksis."*

**Status:** ✓ Kalibrert mot finsk kode.

---

### 2.4 Registrering d – Kraftfôrmengde (`feedAmount`) 📋

**Hva registreres:** Kraftfôr per måltid relativt til hestens kroppsvekt. Grense: 0,3 kg per 100 kg kroppsvekt.

| Verdi | Beskrivelse | Straff |
|---|---|---|
| `none` | Ingen kraftfôr | 0 |
| `within_limit` | Innen anbefalt grense | 0 |
| `over_limit` | Over anbefalt grense | 6 |

**Brukerforklaring:** *"For mye kraftfôr per fôring øker risikoen for kolikk og forfangenhet. Grensen er satt etter finsk protokoll: maks 0,3 kg per 100 kg kroppsvekt per måltid."*

**Status:** ✓

---

### 2.5 Aggregering til C1

```
feedPenalty = straff(feedInterval) + straff(feedOrder) + straff(feedAmount)
C1 = max(0, min(100, bcsScore − feedPenalty))
```

Maks mulig fôrstraff: 100 + 25 + 6 = 131 (kappes effektivt til 100 av `max(0, ...)`).

**Brukerforklaring:** *"Holdet (BCS) er hovedmålet – det viser hvordan hesten faktisk har det. Fôrrutinene gir poengtrekk fordi dårlige rutiner er risikofaktorer som kan føre til problemer over tid, selv om holdet er greit akkurat nå."*

---

## 3. Sub-indeks C2 – Fravær av langvarig tørste

C2 bygges av inntil 4 vann-komponenter som hver gir en delscore 0–100. Komponenter settes til null (utelates) hvis assessor har valgt «Ikke aktuelt».

### 3.1 Registrering e – Vanntilgang inne (`waterIn` + `waterInDuration`) 🏠

| Verdi `waterIn` | Oppfølging `waterInDuration` | Komponentscore |
|---|---|---|
| `yes` (rent vann inne) | – | 100 |
| `no` (ikke rent vann) | `no` (mindre enn 4 t) | 20 |
| `no` | `yes` (mer enn 4 t) | 0 |
| `na` (står ikke inne) | – | (utelates) |

### 3.2 Registrering f – Vanntilgang ute (`waterOut` + `waterOutDuration`) 🏠

| Verdi `waterOut` | Oppfølging `waterOutDuration` | Komponentscore |
|---|---|---|
| `yes` (rent vann ute) | – | 100 |
| `no` | `no` (mindre enn 4 t) | 20 |
| `no` | `yes` (mer enn 4 t) | 0 |

**Foreslått beslutning – manglende "Ikke aktuelt" for vann ute:** Legge til `na`-valg med tekst "Hesten er aldri ute" (f.eks. ved intensiv boksopphold/rehabilitering). Utelates fra C2 på samme måte som `waterIn = na`.

**Brukerforklaring (vann generelt):** *"Vann er livskritisk – en hest uten vann i mer enn 4 timer er i akutt fare. Derfor gir vannmangel over 4 timer score 0, mens kortere mangel gir lav men ikke null score."*

### 3.3 Registrering g – Vannstrøm (`waterFlow`) 🏠

| Verdi | Beskrivelse | Komponentscore |
|---|---|---|
| `high` | God flyt (≥ 8 l/min) | 100 |
| `medium` | Middels (5–8 l/min) | 60 |
| `low` | Lav (< 5 l/min) | 20 |
| `na` | Ikke aktuelt (bøtte, åpent kar) | (utelates) |

### 3.4 Registrering h – Vannpunkter (`waterPoints`) 🏠

| Verdi | Beskrivelse | Komponentscore |
|---|---|---|
| `multiple` | Flere vannpunkter | 100 |
| `single` | Kun ett vannpunkt | 50 |
| `na` | Hesten går alene | (utelates) |

**Brukerforklaring:** *"I grupper kan dominante hester blokkere tilgang til vann. Flere vannpunkter sikrer at alle drikker nok."*

---

### 3.5 Aggregering til C2

Komponentene samles og sorteres stigende. Aggregering med worst-dominance 0,70:

```
Hvis bare 1 komponent:  C2 = komponenten
Hvis 2 eller flere:     C2 = round(worst · 0,70 + gjennomsnitt(resten) · 0,30)
```

**Endring fra forrige versjon:** Vekten endret fra 0,65 til 0,70 for å harmonisere med K3 integument-aggregeringen. Se oversiktsdokument seksjon 2.4 beslutning #2.

**Brukerforklaring:** *"Vannscoren domineres av det svakeste punktet. Har hesten god tilgang inne men ingen ute, trekker det ute-resultatet hele scoren ned. Vann er en helhet – hesten trenger tilgang i alle situasjoner."*

---

## 4. Aggregering til K1 – Choquet-integral

```
mn = min(C1, C2)
mx = max(C1, C2)
μ = 0,12  hvis C1 ≥ C2   (C1 er høyest → dens individuelle vekt)
    0,27  hvis C2 > C1   (C2 er høyest → dens individuelle vekt)
K1 = round(mn · 1,0 + (mx − mn) · μ)
```

**Eksempel:** C1 = 90, C2 = 40 → K1 = 40 + (90−40)·0,12 = **46**.
**Eksempel:** C1 = 40, C2 = 90 → K1 = 40 + (90−40)·0,27 = **54**.

Samme tallpar gir ulik K1 avhengig av *hvilken* komponent som er lav, fordi μ-verdiene er asymmetriske. Det svakeste teller alltid fullt (μ₁₂ = 1,00).

**Brukerforklaring:** *"Ernæringsscoren styres av det svakeste – enten hold eller vann. En hest med utmerket hold men dårlig vanntilgang får lav score, og omvendt. Begge må være på plass for god ernæringsvelferd."*

**Håndtering av manglende sub-indeks:** Hvis kun C1 er registrert: K1 = C1. Hvis kun C2: K1 = C2. Dekningsgrad vises: `K1: 46 (1 av 2)`.

---

## 5. Oppsummeringstabell – alle registreringer

| # | Registrering | Type | Kanal | Maks positiv | Maks negativ |
|---|---|---|---|---|---|
| a | BCS (6 kroppsdeler → flertall) | 🐴 | bcsScore → C1 | 100 (klasse 3–4) | 0 (klasse 1) |
| b | Fôrintervall (timer) | 📋 | feedPenalty → C1 | 0 trekk (≤ 4 t) | 100 trekk (24 t) |
| c | Fôringsrekkefølge | 📋 | feedPenalty → C1 | 0 (grovfôr først) | 25 (kraftfôr først) |
| d | Kraftfôrmengde | 📋 | feedPenalty → C1 | 0 (innen grense) | 6 (over grense) |
| e | Vann inne | 🏠 | C2-komponent | 100 | 0 (manglet > 4 t) |
| f | Vann ute | 🏠 | C2-komponent | 100 | 0 (manglet > 4 t) |
| g | Vannstrøm | 🏠 | C2-komponent | 100 | 20 (< 5 l/min) |
| h | Vannpunkter | 🏠 | C2-komponent | 100 | 50 (kun ett punkt) |

**Pedagogisk tolkning:** BCS er den tyngste enkeltregistreringen – den alene kan flytte scoren fra 0 til 100. Fôringsrekkefølge (maks 25 trekk) og kraftfôrmengde (maks 6 trekk) har relativt liten effekt. Vannmangel er den mest akutte enkeltrisikoen (to registreringer som kan gi 0).

---

## 6. Gjenstående åpne spørsmål (runde 2)

| # | Spørsmål | Foreslått løsning | Status |
|---|---|---|---|
| 1 | BCS klasse 4 severity 0 – riktig? | Beholder. Moderat overvekt er risikofaktor, ikke akutt problem. | ✓ Foreslått |
| 2 | Manglende `na` for vann ute | Legge til. Trivielt i HTML + state. | 🔄 Implementering |
| 3 | Vannpunkter score 50 ved ett punkt – for strengt? | Beholder. Én vannkilde i gruppe er en reell risiko. | ✓ Foreslått |

---

## 7. Referanser i kode

| Element | Linjenummer (ca.) |
|---|---|
| `S`-state initialisering (K1-felter) | 1044–1049 |
| `updateBCS()` – flertallsavstemning | 1175–1200 |
| `wqSpline()` – piecewise kubisk | 1290–1294 |
| `SPLINE.BCS` – koeffisienter | 1299–1303 |
| `CHOQUET_CAPS.K1` – μ-verdier | 1345 |
| `calcK1()` – hele beregningen | 1406–1510 |

HTML-inputfeltene: `<div id="k1" class="page">` rundt linje 329–580.
