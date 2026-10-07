---
name: reiseapp
description: >-
  Oppskrift for å bygge en komplett reiseapp/turplanlegger som én selvstendig
  index.html-fil – samme type app som "Roadtrip Europe 2026". Appen har faner for
  oversikt/"i dag", interaktivt kart, live værvarsel, dagsplan, forslag, pakkeliste
  med avkryssing per person, ladestopp (elbil), delt økonomi/budsjett med
  valutakalkulator, og praktisk info. Ingen bygging, ingen rammeverk – ren
  HTML/CSS/JS med gratis API-er (Open-Meteo, Frankfurter) og delt utgiftsliste via
  Supabase, publisert gratis på Netlify med auto-deploy fra GitHub. Bruk denne
  skillen ALLTID når Janiche vil lage en ny reiseapp, turplanlegger, ferieapp eller
  "app for turen" for en annen reise/tur/ferie – også når hun bare sier "lag en sånn
  app som roadtrip-appen", "en app til sommerferien", "planlegger til interrail-turen"
  e.l. uten å nevne teknologien. Trigger også på: reiseapp, turapp, ferieapp,
  reiseplanlegger, turplanlegger, roadtrip-app, pakkeliste-app, reisebudsjett-app.
---

# Reiseapp – bygg en turplanlegger som én index.html

Denne skillen beskriver nøyaktig hvordan "Roadtrip Europe 2026"-appen er bygd, og
hvordan du lager en ny app for en helt annen tur. Målet er at Janiche kan be om en
ny app på vanlig norsk, og at du kan levere en ferdig, delbar app med samme kvalitet
uten å finne opp hjulet på nytt.

Referanse-appen (den fungerende originalen) ligger som mal i
`assets/index-mal.html`. **Start alltid fra den filen** – kopier den og bytt ut
innholdet. Da arver du all styling, alle API-koblinger og hele strukturen gratis.

## Hva slags app dette er (kjernefilosofien)

- **Én fil.** Alt ligger i `index.html`: HTML, CSS (i `<style>`) og JavaScript (i
  `<script>`). Ingen byggesteg, ingen `npm`, ingen rammeverk. Det gjør appen enkel å
  endre (ett sted), rask å laste, og trivielt å publisere.
- **Mobil først.** Den skal åpnes på telefonen på farten. PWA-aktige meta-tagger gjør
  at den kan legges til på hjemskjermen med eget ikon og se ut som en app.
- **Gratis og nøkkelfri der det går.** Vær (Open-Meteo) og valutakurs
  (Frankfurter) krever ingen API-nøkkel. Kart bruker gratis OpenStreetMap via Leaflet.
- **Delt der det teller.** Felles utgiftsliste ligger i Supabase så alle reisende ser
  det samme på hver sin telefon. Alt annet er statisk innhold i fila.
- **Robust offline.** Henting feiler pent: siste data caches i `localStorage` og vises
  med en «offline»-melding når nettet er borte.
- **Redigeres på norsk.** Hele poenget: Janiche sier hva hun vil endre, Claude Code
  redigerer fila og pusher, Netlify publiserer automatisk på ~30 sekunder.

## Teknologi-stabelen

| Lag | Valg | Hvorfor |
|-----|------|---------|
| Struktur/stil/logikk | Ren HTML + CSS + vanilla JS i én fil | Null bygging, lett å endre, laster raskt |
| Kart | [Leaflet](https://leafletjs.com) 1.9.4 via unpkg CDN + OpenStreetMap-fliser | Gratis, ingen nøkkel |
| Vær | [Open-Meteo](https://open-meteo.com) forecast API | Gratis, ingen nøkkel, god dekning i Europa |
| Valuta | [Frankfurter](https://frankfurter.dev) (ECB-kurser) | Gratis, ingen nøkkel; fast fallback-kurs offline |
| Delt økonomi | [Supabase](https://supabase.com) REST (`/rest/v1`) med publiserbar nøkkel | Gratis nivå, delt mellom telefoner, enkel REST |
| Offline-cache | Nettleserens `localStorage` | Viser sist lagrede data uten nett |
| Hosting | [Netlify](https://netlify.com), auto-deploy fra GitHub | Gratis, publiserer ved hver push |
| Kildekode | GitHub-repo (gren `main`) | Versjonshistorikk + utløser Netlify |

## Fanene i appen

`showTab(navn, knapp)` bytter aktiv fane. Hver fane er en `<div class="section">`.
Referanse-appen har disse (dropp/legg til etter turen):

1. **🗺️ Oversikt** – øverst et «I dag»-kort som automatisk viser dagens etappe +
   dagens vær, eller nedtelling før turen / «velkommen hjem» etter. Deretter nøkkeltall
   (datoer, antall dager, kjørelengde), ferjekort, og reisefølget.
2. **📍 Kart** – Leaflet-kart med alle stopp som fargede markører og kjøreruten som
   stiplet linje. Lastes først når fanen åpnes (`ensureMap()`).
3. **🌤️ Vær** – velg stopp fra nedtrekksliste (eller «📍 Her» for GPS-posisjon),
   henter live varsel fra Open-Meteo.
4. **📅 Dagsplan** – dag-for-dag som trekkspill (`toggleDay()`): klikk en dag for å
   utvide plan, aktiviteter og «husk».
5. **💡 Forslag** – valgfrie aktiviteter/severdigheter per sted.
6. **🎒 Pakkeliste** – avkryssingsliste per person (`data-person`), med
   fremdriftsindikator per person (`updatePersonProgress()`).
7. **⚡ Lading** – ladestopp for elbil langs ruten (dropp hvis bensinbil).
8. **💰 Økonomi** – delt utgiftsliste via Supabase (alle ser samme), fordelt på
   kategorier, pluss en live valutakalkulator.
9. **📋 Praktisk** – forsikring, nødnumre, bookingbekreftelser, nyttig info.

## Slik lager du en ny app for en annen tur

Følg disse stegene. Jobb alltid ut fra `assets/index-mal.html`.

### Steg 1 – Hent inn turinfo (spør Janiche)

Før du bygger, få på plass:

- **Hvem** reiser (navn på hver person → en kolonne i pakkelista og evt. reisefølget)
- **Når** – start- og sluttdato, og evt. faste tider (ferje, fly, henting)
- **Hvor** – reisemål/land (til flagg i header og tittel) og alle stopp i rekkefølge
  med omtrentlige koordinater (lat/lon)
- **Hvordan** – elbil (behold Lading-fanen) eller ikke; transport (ferje, fly, tog)
- **Overnatting** – hoteller/bookinger (booket vs. forslag)
- **Aktiviteter** per dag og valgfrie forslag per sted
- **Økonomi** – hvilke valutaer er aktuelle (til kurser), vil hun ha delt utgiftsliste?
- **Praktisk** – forsikringer, nødnumre, bookingnummer

Koordinater finner du raskt med et nettsøk ("<stedsnavn> latitude longitude") eller
fra et kart. Omtrentlig er godt nok for kart og vær.

### Steg 2 – Grunnoppsett

1. Kopier `assets/index-mal.html` til `index.html` i det nye repoet.
2. Oppdater `<head>`: `<title>`, `apple-mobile-web-app-title`, `theme-color` (hovedfarge),
   og `<html lang="...">` hvis språket skal være et annet.
3. I `<header>`: bytt `h1`-tittel, undertittel, flagg-emojiene i `header::after`, og
   badgene (`.badge`).
4. Evt. juster fargepaletten i `:root`-variablene (`--blue` er hovedfargen).

### Steg 3 – Bytt ut dataene (viktigste jobb)

Alle disse ligger øverst i `<script>`. Endre dem – resten av koden tilpasser seg selv:

| Variabel | Styrer | Fil-sted (ca.) |
|----------|--------|----------------|
| `TRIP_PLACES` | Markører på kartet (navn, `c:[lat,lon]`, farge, popup-tekst) | kart-seksjon |
| `ROUTE` | Kjøreruten (liste med `[lat,lon]`) | kart-seksjon |
| `MARTINE_ROUTE` | Evt. en persons avvikende rute (fjern om unødvendig) | kart-seksjon |
| `WX_STOPS` | Stedene i vær-nedtrekkslista (`n`, `lat`, `lon`, `from`-dato) | vær-seksjon |
| `TRIP_DAYS` | «I dag»-kortet: én rad per dato (`d`, `t`, `lat`, `lon`, `plan`, `husk`) | i dag-seksjon |
| `ECO_RATES` | Valutakurser til NOK for økonomi-summering | økonomi-seksjon |
| `CC_RATE` | Fast fallback-kurs for valutakalkulatoren (NOK per EUR) | valuta-seksjon |
| `SB_URL` / `SB_KEY` | Supabase-prosjektet for delt økonomi | økonomi-seksjon |

Oppdater også selve HTML-innholdet i fanene: dagsplanens trekkspill, pakkelista
(én `.pack-cat`-blokk med `.pitem`-rader per person, hver `<input>` med riktig
`data-person`), forslag, lading og praktisk. I `todayRender()` står det en fast
nedtellings-/«hjemme»-tekst og start-dato (`new Date('ÅÅÅÅ-MM-DD...')`) som må settes
til turens startdato.

Husk å oppdatere initialiseringslinja nederst i `<script>`:
`['janiche','jon',...].forEach(p => updatePersonProgress(p));` → listen må matche
`data-person`-verdiene i den nye pakkelista.

Se `references/teknisk.md` for presise kodemønstre og felt-for-felt-forklaring.

### Steg 4 – Delt økonomi (Supabase)

Hvis turen skal ha delt utgiftsliste, sett opp et nytt Supabase-prosjekt (eller
gjenbruk ett) med en tabell `expenses`:

- `id` (bigint, primærnøkkel, auto)
- `title` (text)
- `category` (text)
- `amount` (numeric)
- `currency` (text)
- `created_at` (timestamptz, default `now()`)

Skru på Row Level Security og lag enkle policyer som tillater `select`, `insert` og
`delete` for `anon`-rollen (dette er en privat delt liste, ikke sensitive data). Lim
inn prosjektets URL i `SB_URL` og den **publiserbare** nøkkelen i `SB_KEY`. Den
publiserbare nøkkelen er laget for å ligge i klienten.

Vil hun ikke ha delt økonomi, fjern Økonomi-fanen (knappen + seksjonen) og de
tilhørende `eco*`-funksjonene, eller la den stå tom.

### Steg 5 – Publiser (GitHub + Netlify)

1. Lag et nytt GitHub-repo og push `index.html` + `netlify.toml` til `main`.
   `netlify.toml` er enkel: ingen bygging, `publish = "."`.
2. I Netlify: «Add new site» → «Import from GitHub» → velg repoet. Siden er statisk,
   så ingen build-kommando trengs.
3. Hver gang noe pushes til `main`, publiserer Netlify automatisk på ~30 sek, samme
   lenke.
4. Lag gjerne en `SLIK-OPPDATERER-APPEN.md` i repoet som forklarer Janiche hvordan hun
   oppdaterer appen (be Claude Code på norsk) – se originalrepoet for mal.

## Når Janiche vil endre en eksisterende reiseapp

Da gjelder samme filosofi: finn riktig datavariabel eller HTML-blokk i `index.html`,
gjør endringen, commit med en kort beskrivende melding, og push til `main`. Hun ser
resultatet live etter ~30 sek. Typiske oppgaver: legge inn/bytte hotell, endre en
dags plan eller tid, legge til et stopp (husk å oppdatere både `TRIP_PLACES`/`ROUTE`
på kartet, `WX_STOPS` og `TRIP_DAYS`), eller justere pakkelista.

## Sjekkliste før du sier deg ferdig

- [ ] Alle datoer, navn, stopp og koordinater stemmer med den nye turen
- [ ] Kartet viser riktige markører og rute (åpne Kart-fanen)
- [ ] «I dag»-kortet viser nedtelling (hvis før turen) og dagens etappe (på en turdag)
- [ ] Pakkelista har riktige personer, og `updatePersonProgress`-init matcher
- [ ] Vær-nedtrekkslista har turens stopp
- [ ] Økonomi: enten fungerende Supabase, eller fanen fjernet/tom – ingen død knapp
- [ ] Valutaer og fallback-kurs passer reisemålet
- [ ] Praktisk-fanen har riktige forsikringer/nødnumre for landene
- [ ] Åpnet på mobilbredde uten horisontal scrolling
