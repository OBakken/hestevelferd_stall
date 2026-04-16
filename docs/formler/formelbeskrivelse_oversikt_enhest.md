# Formelbeskrivelse – oversikt og felles hjelpefunksjoner (en-hest-versjon)

**Gjelder:** `hestevelferd_en_hest.html` (én-hest-versjonen)
**Sist oppdatert:** 2026-04-16
**Status totalt:** 🔄 Runde 1 av revisjon gjennomført – prinsippspørsmål avgjort. Kategori-intern revisjon (runde 2) gjenstår.

> Dette er oversiktsdokumentet for scoring-logikken i én-hest-versjonen av hestevelferds­skjemaet. Det samler de felles hjelpefunksjonene (WQ-spline, Choquet-integral, weighted min-max) som gjenbrukes på tvers av K1–K4, og drøfter prinsippene som er valgt for hele skjemaet.

## Dokumentoversikt

Scoring-logikken er delt i fem dokumenter:

| Dokument | Omfang |
|---|---|
| `formelbeskrivelse_oversikt_enhest.md` (dette) | Felles hjelpefunksjoner, designprinsipper, tverrgående åpne spørsmål |
| `formelbeskrivelse_K1_ernaering_enhest.md` | K1 – Ernæring: BCS, fôr, vann |
| `formelbeskrivelse_K2_oppstalling_enhest.md` | K2 – Oppstalling: bevegelse, liggekomfort, termisk |
| `formelbeskrivelse_K3_helse_enhest.md` | K3 – Helse: skader, smerte, sykdom, bruksrelatert smerte |
| `formelbeskrivelse_K4_atferd_enhest.md` | K4 – Atferd: sosial, beite, menneske-hest, emosjonell |

Hver kategori kan revideres uavhengig av de andre, men flere åpne spørsmål er tverrgående og drøftes her.

---

## Revisjonslogg

Beslutninger tatt i runde 1 (prinsippspørsmål). Alle kategori-dokumenter (K1–K4) må oppdateres i henhold til disse før runde 2 starter.

| # | Spørsmål | Beslutning | Seksjon | Status |
|---|---|---|---|---|
| 1 | Samlet klassifisering K1–K4? | Ja, WQ-regler direkte (Excellent / Enhanced / Acceptable / Not classified) | 1 | 🔄 Implementering |
| 2 | Worst-dominance-vekting – harmoniseres? | Ja, begge settes til 0,70 | 2.4 | 🔄 Implementering |
| 3 | "Fill missing with best" – erstattes? | Ja – minimum-krav + gjennomsnitt + synlig dekningsgrad | 3 | 🔄 Implementering |
| 4 | Risikomålinger – separat status? | Nei, beholdes i score-modellen. Men merkes med målingstype i UI | 4.1 | 🔄 Implementering |
| 5 | Binære registreringer – graderes? | Ja, alle graderes til minst 0/1/2 (følger av beslutning #6) | 4.2 | 🔄 Implementering |
| 6 | Kalibrering mot finsk – egen formel? | Severity-vektet virtuell prevalens med samme WQ-splines som stall | 4.3 | 🔄 Implementering |
| 7 | SPLINE.SKIN og SPLINE.LAMENESS – rydde eller ta i bruk? | Ta i bruk (konsekvens av beslutning #6) | 2.1 | 🔄 Implementering |

**Oppfølging med Essi:** Beslutning #2, #3 og #6 bør bekreftes mot finsk protokoll i neste samtale med Essi. Ingen av dem bryter med WQ-rammeverket slik vi tolker det, men Essi kan ha innsikt i hvordan prevalens-baserte splines best anvendes på én-hest-case som vi bør fange opp før endelig implementering.

---

## 1. Totalscore og overordnet struktur

Skjemaet produserer fire kategoriscore (K1, K2, K3, K4) som hver er et tall mellom 0 og 100. Disse er uavhengige – det finnes **ingen formel som kombinerer K1–K4 til én total­score** i dagens skjema. Hver kategori vises separat, og den samlede bedømmelsen skjer visuelt av assessor.

**Bakgrunn:** I Welfare Quality®-protokollen kombineres kategoriene til en overordnet klassifisering (`Excellent`, `Enhanced`, `Acceptable`, `Not classified`) via regler på tverskategorinivå.

**Beslutning (2026-04-16, Øystein Bakken):** Samlet klassifisering skal legges til. Følger WQ-reglene direkte: Excellent / Enhanced / Acceptable / Not classified. Ingen norsk tilpasning. Rasjonalet: hele skjemaet er bygd for å følge WQ så tett som mulig, og klassifiseringen er den naturlige overbygningen.

**Implementeringsstatus:** 🔄 Spesifikasjonen må hentes inn – de eksakte WQ-tersklene for "Excellent", "Enhanced", osv. er ikke i skjemaet i dag. Kilder: `dairycattleprotocol.pdf` s. 19 (typiske terskler: Excellent = ≥ 55 på alle prinsipper + ≥ 80 på to; Enhanced = ≥ 20 på alle + ≥ 55 på to; Acceptable = ≥ 10 på alle + ≥ 20 på tre; ellers Not classified) eller finsk `Kokohyvinvointiprotokollankuvausliitteineen_final.pdf` for eventuelle horse-spesifikke terskler.

**Oppfølging:** Sjekk med Essi om finsk horse-versjon bruker WQ-tersklene direkte eller har egne. Deretter utarbeid Claude Code-brief.

---

## 2. Felles hjelpefunksjoner

### 2.1 WQ-spline (piecewise kubisk polynom)

All konvertering fra en indeks-verdi I (0–100) til en score (0–100) som skal matche den finske protokollen, bruker en piecewise kubisk polynom-funksjon:

```
Hvis I ≤ knot:
    score = a_low + b_low·I + c_low·I² + d_low·I³
Hvis I > knot:
    score = a_high + b_high·I + c_high·I² + d_high·I³
Klipp til [0, 100]
```

**Hvor den brukes (etter revisjon):**

| Spline | Knot | Brukes i |
|---|---|---|
| `BCS` | 80 | K1 – holdvurdering |
| `COMFORT` | 62 | K2 – strøkvalitet (bedding) |
| `DISEASE` | 62 | K3 – sykdomsalarm |
| `PASTURE` | 50 | K4 – beitedager |
| `HUMAN_ANIMAL` | 70 | K4 – VAA/FAA-resultat |
| `SKIN` | 65 | K3 – hudlesjoner, hevelser, hoof, scratch (skal tas i bruk) |
| `LAMENESS` | 78 | K3 – halthet (skal tas i bruk) |

**Beslutning (2026-04-16, Øystein Bakken):** `SPLINE.SKIN` og `SPLINE.LAMENESS` skal tas i bruk som del av severity-vektet virtuell prevalens-logikken (se seksjon 4.3). De gjør at én-hest-versjonen bruker nøyaktig samme kurveform som stall-versjonen, bare med N = 1 og severity-vekting som input.

**Implementeringsstatus:** 🔄 Endringene kommer som en del av refaktoreringen i beslutning #6 (seksjon 4.3). Aktiveres i `calcK3()`.

---

### 2.2 WQ Choquet-integral

Choquet-integralen er en generalisering av et vektet gjennomsnitt som tillater at par og grupper av sub-indekser kan veies forskjellig fra summen av enkeltvektene. Dette modellerer ikke-additive interaksjoner: f.eks. at "alt må være bra samtidig" (superadditivitet) eller at "det verste dominerer" (subadditivitet).

**Kapasitetsstruktur:** For n sub-indekser trengs 2ⁿ − 2 kapasiteter (μ-verdier), én for hver mulig undermengde (unntatt den tomme og den fulle – sistnevnte er alltid 1).

| Kategori | Antall sub-indekser | Antall kapasiteter |
|---|---|---|
| K1 | 2 | 2 (μ₁, μ₂) |
| K2 | 3 | 6 (μ₁, μ₂, μ₃, μ₁₂, μ₁₃, μ₂₃) |
| K3 | 4 | 14 (μ₁–μ₄, μ₁₂, μ₁₃, …, μ₂₃₄) |
| K4 | 4 | 14 |

Pluss den fulle mengden (μ₁…ₙ = 1,00) gir dette totalt 15 + 15 = 30 finske kapasitetsverdier for K3 og K4.

**Algoritmen (`wqChoquet`):**

```
Sorter sub-indeksene stigende etter score.
Result = 0
For hver score i sortert rekkefølge:
    diff = score − forrige score (eller 0 for første)
    remaining = mengden av indekser fra og med denne
    Result += diff · μ(remaining)
Klipp til [0, 100]
```

**Generell tolkning:**
- Det laveste elementet teller alltid med μ_full (= 1,00) – altså 100 %.
- Hver høyere score bidrar bare med den delen som *overstiger* forrige, multiplisert med kapasiteten til gruppen som gjenstår.
- Hvis alle par-/tripp-μ-er er store (nær 1), blir resultatet nær maks(scoene).
- Hvis alle enkelt-μ-er er små, blir resultatet nær min(scoene).

**Implikasjon for revisjon av vekter:** Det er ikke tilstrekkelig å endre enkelt-μ-verdier uten å også vurdere par- og tripp-μ-ene. De er matematisk koblet via Möbius-transformen og avhenger av faglig syn på hvordan sub-indekser interagerer (om de er substitutter eller komplementer).

**Status:** ✓ `wqChoquet`-funksjonen er matematisk korrekt. Kapasitetene er hentet fra finsk protokolls Taulukko 18 (for K3) og tilsvarende tabeller for K1, K2, K4.

**Kommentar til revisjon:** [tom – bør kapasitetene valideres av en fagperson med matematisk kompetanse i Choquet-integraler? Tas opp i runde 3 av revisjonen.]

---

### 2.3 K1's to-variabel Choquet (håndkodet)

K1 bruker IKKE den generelle `wqChoquet`-funksjonen, men en håndkodet versjon for to variable:

```
mn = min(C1, C2)
mx = max(C1, C2)
μ = μ for den med høyere score (som "alene")
K1 = round(mn·1,0 + (mx − mn)·μ)
```

**Matematisk** gir dette samme resultat som `wqChoquet` for to variable. Det er duplisert logikk, men ikke funksjonelt annerledes.

**Status:** 🔄 Kode-duplisering. Bør refaktoreres til å bruke `wqChoquet` også i K1 for konsistens. Lavprioritet – kan tas samtidig med andre implementeringsendringer i en Claude Code-økt.

---

### 2.4 Weighted min-max (worst dominates)

En tredje aggregeringsform brukes for sub-indekser med få komponenter som ikke har Choquet-kapasiteter definert:

```
Sorter komponentene stigende.
worst = komponenten med lavest score.
rest = de andre.
resultat = round(worst · vekt_worst + gjennomsnitt(rest) · (1 − vekt_worst))
```

**Hvor det brukes (etter revisjon):**

| Hvor | vekt_worst | Kommentar |
|---|---|---|
| K1 – C2 vann | 0,70 | Harmonisert |
| K3 – Idx1 integument | 0,70 | Uendret |

**Beslutning (2026-04-16, Øystein Bakken):** Vektene harmoniseres til 0,70 for begge aggregater (K1-C2 vann og K3-Idx1 integument). Ingen faglig begrunnelse for differensiering ble identifisert; 0,70 gir moderat-sterk worst-dominance som er konsistent med at kritiske enkeltfunn skal dominere domenescoren.

**Implementeringsstatus:** 🔄 Trivielt kodeendring i `calcK1()` (C2-aggregering). K3 beholder eksisterende verdi.

**Oppfølging:** Diskuteres med Essi ved neste møte for bekreftelse av at 0,70 er i tråd med hennes tolkning av protokollen.

---

### 2.5 Min-aggregering (absolutt worst)

I K3's Idx2 (annen smerte) brukes ren `min()`:

```
otherPainScore = min(hgsScore, backScore)
```

Ingen interaksjon, ingen kompromiss. Dette er den strengeste formen for aggregering i skjemaet, og brukes bare for smerteindikatorer der WQ-protokollen foreskriver "worst completely dominates".

**Status:** ✓ I tråd med finsk protokoll.

---

## 3. Håndtering av manglende sub-indekser (tidligere "fill missing with best")

**Bakgrunn:** Dagens heuristikk fyller manglende sub-indekser med maksimum av de tilgjengelige. Dette gir en optimistisk (ikke konservativ) antagelse og skjuler ufullstendighet. Problemet gjelder særlig i K2, K3 og K4 der Choquet-beregning krever at alle sub-indekser har verdier.

**Beslutning (2026-04-16, Øystein Bakken):** "Fyll med best"-heuristikken erstattes med en tredelt mekanisme:

1. **Minimum-krav per kategori.** Kategoriscoren beregnes som numerisk verdi kun hvis minimum er oppfylt:
   - K1 (2 sub-indekser): krever minst 1 av 2
   - K2 (3 sub-indekser): krever minst 2 av 3
   - K3 (4 sub-indekser): krever minst 3 av 4
   - K4 (4 sub-indekser): krever minst 3 av 4
   - Under minimum: vises "Ufullstendig" med liste over manglende sub-indekser.

2. **Gjennomsnitt, ikke maksimum, som fyllverdi** når minimum er oppfylt men noe fortsatt mangler. Dette er en balansert antagelse som verken over- eller undervurderer ukjent status.

3. **Synlig dekningsgrad** i kategorioverskriften: f.eks. `K3: 78 (3 av 4 sub-indekser registrert)`. Gjelder ALLTID, også når alle sub-indekser er registrert (da vises `K3: 78 (4 av 4)`).

**Implementeringsstatus:** 🔄 Claude Code-brief må utarbeides. Påvirker `calcK2`, `calcK3`, `calcK4` og UI-visningene for kategoriscoren.

**Oppfølging:** Essi konsulteres for å bekrefte at dette ikke bryter med finsk protokolls håndtering av manglende data. Hvis finsk protokoll har en spesifikk regel, harmoniseres i stedet med den. Mest sannsynlig behandler finsk protokoll manglende data strengere enn dagens heuristikk, siden deres N-baserte utvalg krever fullstendig datasett for score-beregning.

---

## 4. Tverrgående designprinsipper

### 4.1 Utfallsmåling vs. risikomåling

Skjemaet blander **utfallsmålinger** (tegn på hvordan hesten faktisk har det nå) og **risikomålinger** (strukturelle forhold som *kan* påvirke velferden):

| Utfall (animal-based) | Risiko/miljø (resource- eller management-based) |
|---|---|
| BCS (faktisk hold) | Fôrintervall (rutine) |
| HGS (ansiktssmerte) | Tid på boks (struktur) |
| VAA/FAA (responsmønster) | Tilgang til ly (struktur) |
| Stereotypier (atferd) | Berikelse (tilbud) |
| Halthet (funksjon) | Ressursplassering (mulighet) |

**Beslutning (2026-04-16, Øystein Bakken):** Risikomålinger (ressurs- og styringsbaserte) skal beholdes innenfor score-modellen, i tråd med WQ-protokollen og finsk praksis. De skal ikke flyttes til en separat advarsels-kanal utenfor scoren.

**Kildegrunnlag for beslutningen:** WQ-protokollen (`dairycattleprotocol.pdf` s. 6 og 13) er klar på at animal-based measures foretrekkes, men at resource/management-based målinger brukes når animal-based ikke er tilgjengelig eller ikke er sensitiv nok. Finsk protokoll (`Kokohyvinvointiprotokollankuvausliitteineen_final.pdf` s. 9) følger samme prinsipp. Ingen av disse skiller risikomålinger ut i separat kanal – alle målinger inngår i samme scorestruktur.

**Supplerende tiltak:** Hvert spørsmål i skjemaet skal merkes tydelig med målingstype (Eläinperäinen / Resurssiperäinen / Toimitapamittari ≈ dyrbasert / ressursbasert / styringsbasert), slik finsk protokoll gjør på `hyvinvointikortit`. Dette er en UI-forbedring som gir transparens uten å bryte WQ-strukturen.

**Implementeringsstatus:** 🔄 UI-endring i `hestevelferd_en_hest.html` – legge til små merker/ikoner ved hver registrering. Ingen endring i scoreberegning. Forslag til ikoner:
- 🐴 Dyrbasert (observasjon av hesten)
- 🏠 Ressursbasert (tilstedeværelse av ressurs)
- 📋 Styringsbasert (rutine, intervju)

**Referanse:** WQ dairy protocol v3.2 s. 6 og 13; AWIN 2015 s. 9; Kokohyvinvointiprotokollankuvausliitteineen_final.pdf s. 9; `hyvinvointikortit_saavutettava.pdf`.

---

### 4.2 Binære vs. graderte registreringer

Skjemaet har i dag en klar over­vekt av binære registreringer (ja/nei), særlig i K3 (helse):

| Binær registrering (i dag) | Foreslått gradering (etter revisjon) |
|---|---|
| Halthet (ja/nei) | 0 = ingen / 1 = mild (kun trav) / 2 = alvorlig (synlig i skritt) |
| Ryggsmerte (ja/nei) | 0 = ingen / 1 = mild respons ett punkt / 2 = kraftig eller flere punkter |
| Hovproblemer (ja/nei) | 0 = ingen / 1 = én hov eller mild / 2 = flere hover eller alvorlig |
| Utstyrsskader (ja/nei) | 0 = ingen / 1 = mild gnag / 2 = åpent sår eller stor skade |
| Gnagemerker (ja/nei) | 0 = ingen / 1 = lett / 2 = tydelig slitasje |
| Hoste (ja/nei) | 0 = ingen / 1 = sporadisk / 2 = gjentakende |
| Diaré (ja/nei) | 0 = ingen / 1 = løs avføring / 2 = tilsølt bakpart |
| Kolikk siste 12 mnd (ja/nei) | 0 = ingen / 1 = én episode / 2 = behandlet eller gjentakende |
| Tilgang til ly (ja/nei) | 0 = ingen / 1 = delvis (takutspring, tett skog) / 2 = godkjent |

**Beslutning (2026-04-16, Øystein Bakken):** Alle binære registreringer graderes til minst 0/1/2. Dette følger automatisk av hovedbeslutningen om severity-vektet virtuell prevalens (seksjon 4.3), som forutsetter at målinger er graderte for å gi meningsfull oppløsning.

**Implementeringsstatus:** 🔄 Endringer i HTML (ny valgstruktur per måling) og i tilhørende state-håndtering i JS. Hver gradering trenger faglig forsvarlig beskrivelse av kriteriene – tas i kategori-spesifikke dokumenter (K1–K4).

**Oppfølging:** Graderingstekstene bør gjennomgås med Christien og eventuelt veterinærfaglig ressurs for klinisk presisjon. Særlig viktig for halthet (AAEP-skala eller forenklet versjon?) og mouth/tack hvor det finnes etablerte graderinger i finsk protokoll.

---

### 4.3 Kalibrering mot finsk protokoll

Finsk protokoll er bygd for stallvurdering og bruker prevalens i en gruppe hester som input til WQ-splinene. For én hest kan prevalens bare være 0 % eller 100 %, som gjør at mange målinger kollapser til binære utfall hvis prevalens-formelen brukes direkte.

**Beslutning (2026-04-16, Øystein Bakken):** Én-hest-versjonen skal bruke **severity-vektet virtuell prevalens** med samme WQ-spline-koeffisienter som stall-versjonen. Alle binære målinger graderes til minst 0/1/2 (se seksjon 4.2). Det innføres ikke separate spline-koeffisienter for én-hest-versjonen.

**Formel:**

```
virtualPrev = (count_mild · 0,5 + count_severe · 1,0) / N      (N = 1 for én hest)
I = 100 − virtualPrev · 100
score = WQ-spline(I, knot, koeffisienter)   ← samme som stall
```

**Eksempler med N = 1:**

| Observasjon | virtualPrev | I | Score (ca., avhenger av spline) |
|---|---|---|---|
| 0 = ingen funn | 0 | 100 | 100 |
| 1 = mild funn | 0,5 | 50 | 30–50 (spline-avhengig) |
| 2 = alvorlig funn | 1,0 | 0 | 0 |

**Rasjonale:** Dette speiler logikken finsk protokoll allerede bruker for `mouth`, `saddling` og `hgs` – der mild teller 0,5 og alvorlig teller 1,0. Ved å generalisere dette til alle målinger får én-hest-versjonen:
- Samme spline-kurver som stall-versjonen (ingen ny kalibrering)
- Konsistent severity-vekting på tvers av målinger
- Mulighet for at SPLINE.SKIN og SPLINE.LAMENESS kan brukes slik de er kalibrert

**Sammenlignbarhet mellom stall og en-hest:** Metodikken er identisk, men tallene er ikke direkte sammenlignbare. En stall med 5 % alvorlig halte hester gir LAMENESS virtualPrev = 0,05 → I = 95 → høy score. En enkelthest med mild halthet gir virtualPrev = 0,5 → I = 50 → lavere score. Begge er meningsfulle i sin kontekst, men uttrykker ulike ting: gruppeprevalens kontra individuell alvorlighet.

**Kommunikasjon i rapporten (ikke i skjemaet):** Rapporten som genereres etter en vurdering skal inneholde en eksplisitt merknad om dette. Forslag til rapport-tekst:

> "Denne vurderingen er basert på observasjon av én hest på ett tidspunkt. Score-verdiene indikerer den individuelle hestens velferdssituasjon og bruker samme WQ-rammeverk som stallvurderinger, men med severity-vektet virtuell prevalens for N = 1. Tallverdiene kan derfor ikke direkte sammenlignes med score fra gruppe-/stallvurderinger, som er statistisk mer robuste. Enkeltobservasjoner (særlig HGS og VAA/FAA) kan være påvirket av situasjonsfaktorer; gjenta gjerne vurderingen hvis score virker uventet lav."

**Implementeringsstatus:** 🔄 Omfattende refactoring av `calcK1`–`calcK4`. Dette er den største enkelte endringen i revisjonen. Følgende steg i rekkefølge:

1. Gradér binære målinger i HTML (seksjon 4.2)
2. Oppdater state-objektet `S` for å lagre graderte verdier
3. Skriv generisk hjelpefunksjon `severityToScore(mildCount, severeCount, N, spline)` som innkapsler virtualPrev-formelen
4. Refaktorér `calcK3` først (flest affekterte målinger), deretter K2 og K4
5. K1 berøres i hovedsak av beslutning #2 (vann-vekting) og gjenværende severity-konvertering av BCS (som allerede følger logikken)
6. Kjør regresjonstester: test én-hest-case med kjent severity mot stall-case med kjent prevalens, sjekk at begge gir forventede score.

**Iboende begrensninger som gjenstår:**

- Én hest kan aldri gi data om *systemproblemer i stallen* – f.eks. at "mange hester har mildt skorter skulder" er et systemsignal som ikke fanges opp
- Enkelt-HGS er mer usikkert enn flokk-gjennomsnitt – én dårlig dag kan gi urepresentativ score
- VAA/FAA på én hest er stokastisk – én dårlig test gir stort utslag

Disse må kommuniseres i rapporten, men de endrer ikke selve scoreberegningen.

**Oppfølging:** Essi bør konsulteres for å bekrefte at severity-vektene (0,5 / 1,0) er i tråd med protokollens ånd, og for å validere graderingsnivåene for de binære målingene som skal graderes.

---

## 5. Gjenstående åpne spørsmål

Etter runde 1 (prinsippspørsmål) gjenstår følgende åpne spørsmål for runde 2 (kategori-intern revisjon) og runde 3 (Choquet-kapasiteter):

### Gjelder enkeltkategorier (runde 2)

1. **Ubrukt `tester`-variabel i K4** – en konkret inkonsistens. `S.tester` lagres, men brukes ikke i beregning. Skal kjent tester gi en score-reduksjon, flagges som usikker måling, eller fjernes?

2. **`maxAlarms = 7` i K3** bør være 7,5 for å matche faktiske maks verdier (1,0 + 1,0 + 1,0 + 1,5 + 3,0). Trivial bug eller bevisst forenkling? Bekreftes mot finsk kode.

3. **Lineær HGS-skalering** bryter mønster fra andre spline-baserte målinger. Bør HGS konverteres til WQ-spline-basert (via virtualPrev-formelen i 4.3)? Sannsynligvis ja, men avhenger av om det finnes en HGS-spesifikk spline i finsk protokoll.

4. **Severity-mapping for integument-funn** (skin, swelling, hoof, scratch) – spesielt forholdet mellom skin=1 (65) og hoof=yes (25) bør diskuteres faglig. Når målingene graderes til 0/1/2, skal mapping til virtualPrev være ensartet (0, 0,5, 1,0) eller differensiert etter funn-type?

5. **Kolikk historisk vs. akutt.** Kolikk siste 12 måneder gir 3 alarm-poeng – mer enn to alvorlige pågående funn. Bør graderes (som i beslutning #5) og vektlegges mindre? Eller skilles ut som egen historikk-kanal?

6. **mgmtPainScore-formelen i K3** (vektet kombinasjon med intern vekt 2,0 / 1,0 / 1,5) er ad-hoc og kompleks. Bør erstattes med enklere regel, evt. Choquet med tre kapasiteter.

7. **Scan-observasjonens plassering og bruk (K2)** – observeres under "Bevegelsesfrihet" men brukes i "Liggekomfort"-beregning. Avklar hvor den skal ligge og hvordan den skal brukes.

8. **μ₂ (beite) = 0,07 i K4** er lavest av alle enkelt-μ. For norske forhold virker dette kontraintuitivt. Tas opp i runde 3.

### Tverrgående (runde 3 / matematisk validering)

9. **Validering av Choquet-kapasiteter** av fagperson med matematisk kompetanse. Alle kapasitetsverdier er hentet fra finsk protokoll, men Möbius-struktur og interaksjonsvekting bør sjekkes.

10. **Kode-duplisering K1-Choquet vs. `wqChoquet`.** Refaktorering for konsistens. Lavprioritet.

11. **Utestede ekstremverdi-konstellasjoner.** Hva skjer ved alle 0 i én kategori og alle 100 i en annen? Bør testes systematisk i runde 4.

---

## 6. Anbefalt videre revisjonsflyt

**Runde 1 – prinsippspørsmål først (dette dokumentet):** ✓ **Fullført 2026-04-16.** Samtlige tverrgående beslutninger tatt (se revisjonslogg øverst).

**Runde 2 – kategori-intern revisjon (K1 → K2 → K3 → K4):** 🔄 **Neste steg.** For hver kategori:
- Oppdater kategori-dokumentet med de vedtatte prinsippbeslutningene (særlig gradering av binære målinger og severity-vektet virtuell prevalens)
- Svar på kategori-spesifikke åpne spørsmål (se seksjon 5 over)
- Faglig revisjon av graderings­tekster, severity-mapping, og interne vekter

**Runde 3 – Choquet-kapasiteter:** 🔄 **Krever Essi.** Spesielt μ-rangering for K3 (μ₂ = annen smerte) og K4 (μ₄ = emosjonell, μ₂ = beite). Dette krever fagfolk med kunnskap om finsk protokoll.

**Runde 4 – valideringskjøring:** 🔄 Når alle endringer er spesifisert i MD, kjør testdata gjennom ny og gammel versjon for å se hvilke hester som endrer score, og hvor mye. Bekrefter at revisjonen gir forventede effekter før implementering.

**Runde 5 – implementasjon via Claude Code:** 🔄 Når endringene er validert, skrives én eller flere briefer som Claude Code bruker til å oppdatere `hestevelferd_en_hest.html`. Parallelt vurderes om stall-versjonen skal få samme endringer (særlig for beslutning #2, #3 og #4 som også gjelder der).

---

## 7. Referanser i kode

| Element | Linjenummer (ca.) |
|---|---|
| `S`-state initialisering (alle kategorier) | 1044–1061 |
| `updateBCS()` | 1175–1200 |
| `setScan()` | 1203–1221 |
| `handleEnrichment()` | 1223–1235 |
| `updateSaddling()` | 1238–1255 |
| `updateHGS()` | 1258–1270 |
| `selectScenario()` / `selectVAA()` | 1273–1284 |
| `wqSpline()` | 1290–1294 |
| `SPLINE` – alle koeffisienter | 1297–1340 |
| `CHOQUET_CAPS` – alle μ-verdier | 1343–1367 |
| `wqChoquet()` | 1370–1387 |
| `weightedMinMax()` | 1390–1400 |
| `calcK1()` | 1406–1510 |
| `calcK2()` | 1515–1661 |
| `calcK3()` | 1664–1822 |
| `calcK4()` | 1825–1915 |
