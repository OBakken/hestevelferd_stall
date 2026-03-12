# Hestevelferd – Welfare Quality® vurderingsverktøy

Digitalt verktøy for vurdering av hestevelferd basert på den finske Welfare Quality®-protokollen, tilpasset norske forhold.

## Om prosjektet

Verktøyet er utviklet i samarbeid med [Norsk Hestesenter](https://www.norsk-hestesenter.no/) som del av et nordisk samarbeid med det finske forskningsmiljøet i Kaustinen og svenske Wången (Interreg).

Skjemaet dekker fire hovedkategorier:
- **K1 – God ernæring:** Holdvurdering (BCS), fôring og vanntilgang
- **K2 – Godt oppstallingsmiljø:** Bevegelsesfrihet, liggekomfort, renhet og termisk komfort
- **K3 – God helse:** Kliniske funn, munnundersøkelse, kolikk, påsaling, HGS (smertevurdering)
- **K4 – Hensiktsmessig atferd:** Sosial kontakt, beite, stereotypier, menneske-dyr-relasjoner

## Bruk

Åpne `Skjema_hestevelferd_stall.html` i en nettleser. Alt kjører lokalt — ingen data sendes noe sted. Vurderingsdata kan lagres i nettleserens localStorage og eksporteres for verifisering mot den finske referansesiden.

## Mappestruktur

```
hestevelferd/
├── README.md
├── .gitignore
├── Skjema_hestevelferd_stall.html
└── docs/
    ├── How_the_welfare_assessment_calculation_system_works.docx
    ├── Beregning_av_vann.docx
    └── Håndbok_hestevelferdsindikatorer_oversatt.docx
```

## Beregningsmodell

Verktøyet implementerer den finske beregningsmodellen med spline-transformasjoner og Choquet-integraler for å aggregere delindekser til kategori- og totalscorer. Beregningene er kalibrert mot den finske referansenettsiden (Pisteiden laskenta).

Se `docs/How_the_welfare_assessment_calculation_system_works.docx` for detaljer om beregningslogikken.

### Kjente begrensninger

- **Sosial atferd (C2):** Proxy-formel for den finske logikken som belønner «rolige flokker». Krever ytterligere kalibrering.
- **Veiledende scorer:** Beregningene er veiledende og kan avvike noe fra offisiell finsk Choquet-aggregering.

## Referanser

- Wageningen UR Livestock Research (2011). *Welfare Monitoring System – Assessment protocol for horses*, version 2.0. Report 569.
- [AWIN Protocol for Horses](https://air.unimi.it/retrieve/handle/2434/269097/384836/AWINProtocolHorses.pdf)
- Finsk hestevelferdsprotokoll (Kokohyvinvointiprotokollankuvausliitteineen)
- [HGS Guide 2025](https://www.vetmed.ucdavis.edu/horse-grimace-scale)

## Lisens

Utviklet for Norsk Hestesenter. Kontakt prosjektleder for bruksvilkår.
