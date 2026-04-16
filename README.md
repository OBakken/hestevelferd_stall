# Hestevelferd – digitale vurderingsverktøy

Prosjektet utvikler digitale verktøy for vurdering av hestevelferd, basert på den finske Welfare Quality-protokollen utviklet av Essi Wallenius. Prosjektet er en del av et nordisk Interreg-samarbeid mellom Norsk Hestesenter, Kaustinen (Finland) og Wangen (Sverige).

## Filer i repoet

**hestevelferd_en_hest.html** – En-hest-versjonen, for vurdering av enkelthester. Bruker severity-vektet virtuell prevalens for N=1. Revidert 2026-04-16 (se docs/formler/).

**Skjema_hestevelferd_stall.html** – Stall-versjonen, for vurdering av en hel besetning. Bruker prevalens-basert scoring i trad med finsk protokoll. Ikke pavirket av en-hest-revisjonen.

**hestevelferd_skjema_Finsk.html** – Referansefil fra finsk protokoll. Brukes som fasit for scoring-logikk. Skal ikke redigeres.

**index.html** – Redirect til stall-versjonen.

## Dokumentasjon

**docs/formler/** inneholder komplette formelbeskrivelser for en-hest-versjonen:

- `formelbeskrivelse_oversikt_enhest.md` – prinsipper, felles hjelpefunksjoner, revisjonslogg
- `formelbeskrivelse_K1_ernaering_enhest.md` – BCS, for, vann
- `formelbeskrivelse_K2_oppstalling_enhest.md` – bevegelse, liggekomfort, termisk
- `formelbeskrivelse_K3_helse_enhest.md` – skader, smerte, sykdom
- `formelbeskrivelse_K4_atferd_enhest.md` – sosial, beite, menneske-hest, emosjonell
- `claude_code_brief_revisjon_enhest.md` – implementasjonsspesifikasjon

Tilsvarende dokumentasjon for stall-versjonen eksisterer ikke enna.

**Referansedokumenter i roten:**

- `How_the_welfare_assessment_calculation_system_works.docx` – oversikt over den finske beregningsmodellen
- `Beregning_av_vann.docx` – detaljert vannscoring
- `Handbok_hestevelferdsindikatorer_oversatt.docx` – oversatt handbok
- `K3_feilretting_brief.md` – feilrettingsdokumentasjon for K3
- `AWINProtocolHorses.pdf` – AWIN-protokoll for hest
- `HGS_GUIDE2025.pdf` – Horse Grimace Scale veiledning
- `Koko-hyvinvointiprotokollan-kuvaus-liitteineen_final.pdf` – komplett finsk protokoll
- `welfare_monitoring_system_assessment_protocol_for-wageningen_university_and_research_238619.pdf` – Wageningen-protokoll

## Viktig: ulike kodebaser

En-hest-versjonen og stall-versjonen deler konseptuelt rammeverk (WQ-protokollen), men har separate scoring-strukturer:

- Stall-versjonen bruker prevalens i flokk som input til WQ-splines
- En-hest-versjonen bruker severity-vektet virtuell prevalens med N=1

Endringer i en fil skal ikke automatisk speiles i den andre. Enhver parallell revisjon ma vurdere tilpasninger til den aktuelle kodebasen.

## Teknisk

Hver HTML-fil er en selvstendig applikasjon (inline CSS og JS, ingen avhengigheter). Kan apnes direkte i nettleseren eller serveres med en enkel lokal server (f.eks. `python -m http.server 8000`). Data lagres i nettleserens localStorage.

## Revisjonshistorikk

En-hest-versjonen revidert 2026-04-16 med 7 prinsippbeslutninger: samlet WQ-klassifisering, harmonisert worst-dominance (0,70), minimum-krav og fill-with-avg for manglende data, malingstype-merking, gradering av binare malinger, severity-vektet virtuell prevalens med WQ-splines, aktivering av SPLINE.SKIN. Detaljer i `docs/formler/formelbeskrivelse_oversikt_enhest.md`.

## Referanser

- Wageningen UR Livestock Research (2011). *Welfare Monitoring System – Assessment protocol for horses*, version 2.0. Report 569.
- AWIN Protocol for Horses (2015).
- Finsk hestevelferdsprotokoll – Koko hyvinvointiprotokollan kuvaus liitteineen.
- Horse Grimace Scale (HGS) Guide 2025.

## Kontakt

Norsk Hestesenter. Prosjektansvarlig: Gunn Elisabeth Skullerud. Utvikler: Oystein Bakken.
