# Teknisk referanse – reiseapp

Presise kodemønstre fra `assets/index-mal.html`. Les dette når du skal bytte ut data
eller skjønne hvordan en fane fungerer. Linjenumre er omtrentlige (fila er ~1750 linjer).

## Innhold
- [Grunnoppsett i `<head>`](#grunnoppsett-i-head)
- [Designsystem (CSS-variabler)](#designsystem-css-variabler)
- [Faneveksling](#faneveksling)
- [Kart (Leaflet)](#kart-leaflet)
- [«I dag»-kort (TRIP_DAYS)](#i-dag-kort-trip_days)
- [Vær (Open-Meteo)](#vær-open-meteo)
- [Delt økonomi (Supabase)](#delt-økonomi-supabase)
- [Valutakalkulator (Frankfurter)](#valutakalkulator-frankfurter)
- [Pakkeliste per person](#pakkeliste-per-person)
- [Dagsplan-trekkspill](#dagsplan-trekkspill)
- [Initialisering nederst](#initialisering-nederst)

## Grunnoppsett i `<head>`

PWA-aktige tagger gjør at appen kan legges på hjemskjermen som en app:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<meta name="theme-color" content="#1a4fa0">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="Roadtrip 2026">
<title>🚗 Roadtrip Europe 2026 – Sjurseth-Eliassen</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" ...>
```

Bytt `theme-color` til hovedfargen, `apple-mobile-web-app-title` til kort appnavn
(vises under ikonet), og `<title>`.

## Designsystem (CSS-variabler)

Alle farger og mål defineres som variabler i `:root`, så du endrer stilen ett sted:

```css
:root {
  --blue:#1a4fa0; --blue-l:#2e6dd4; --blue-pale:#e8f0fb;  /* hovedfarge */
  --green:#1a7a4a; --green-l:#e6f4ec;
  --orange:#d4650a; --orange-l:#fdf0e6;
  --red:#b91c1c; --red-l:#fef2f2;
  --purple:#7c3aed; --purple-l:#fdf4ff;
  --gray:#374151; --gray-l:#f3f4f6; --gray-m:#9ca3af;
  --shadow:0 2px 12px rgba(0,0,0,0.08); --r:14px;  /* skygge + hjørneradius */
}
```

Gjenbrukbare klasser: `.card` (hvitt kort), `.igrid`/`.icard` (nøkkeltall-rutenett),
`.badge` (pille i header), `.sec-title` (seksjonsoverskrift), `.alert` (info/varsel).
Rutenett kollapser til én kolonne på smal skjerm via `@media(max-width:480px)`.

## Faneveksling

```javascript
function showTab(name, btn) {
  document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
  document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
  document.getElementById(name).classList.add('active');
  btn.classList.add('active');
  window.scrollTo({top:0, behavior:'smooth'});
  if(name === 'kart') ensureMap();          // kart lastes først ved åpning
  if(name === 'okonomi'){ ecoFetch(); ccFetchRate(); }
  if(name === 'vaer') wxLoad();
}
```

Knapp: `<button class="tab" onclick="showTab('kart',this)">📍 Kart</button>`
Seksjon: `<div id="kart" class="section"> ... </div>`
`id` på seksjonen må matche første argument i `showTab`.

## Kart (Leaflet)

```javascript
var TRIP_PLACES = [
 {n:'Slependen (hjem)', c:[59.889,10.523], col:'#1a7a4a', d:'🏠 Start & slutt'},
 {n:'Aalborg', c:[57.0488,9.9217], col:'#1a7a4a', d:'🏨 B&B Hotel – natt til 7/7'},
 // ... ett objekt per stopp. n=navn, c=[lat,lon], col=markørfarge, d=popup-HTML
];
var ROUTE = [[59.889,10.523],[59.0537,10.0357], /* ... */];  // kjørelinje i rekkefølge

var tripMap = null;
function ensureMap(){
  if(tripMap){ setTimeout(function(){ tripMap.invalidateSize(); }, 60); return; }
  tripMap = L.map('map', {scrollWheelZoom:false}).setView([54.5,8.5], 5);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
    {maxZoom:18, attribution:'© OpenStreetMap'}).addTo(tripMap);
  L.polyline(ROUTE, {color:'#1a4fa0', weight:3, opacity:.5, dashArray:'6,8'}).addTo(tripMap);
  TRIP_PLACES.forEach(function(p){
    L.circleMarker(p.c, {radius:9, color:'#fff', weight:2, fillColor:p.col, fillOpacity:1})
      .addTo(tripMap).bindPopup('<b>'+p.n+'</b><br>'+p.d);
  });
  tripMap.fitBounds(TRIP_PLACES.map(function(p){return p.c;}), {padding:[25,25]});
}
```

`d`-feltet kan inneholde HTML, f.eks. en «Åpne i kart»-lenke:
`<a href="https://www.google.com/maps/dir/?api=1&destination=..." target="_blank">Åpne i kart ↗</a>`.
`invalidateSize()` trengs fordi kartet rendres i en skjult fane til første åpning.

## «I dag»-kort (TRIP_DAYS)

Driver det blå kortet øverst på Oversikt. Én rad per dato:

```javascript
var TRIP_DAYS = [
 {d:'2026-07-06', t:'Slependen → Larvik → ferje → Aalborg', lat:57.05, lon:9.92,
  plan:'Kjør til Larvik, ferje 17:30 → Hirtshals 21:15, videre til Aalborg.',
  husk:'CEU1949-billett + pass klare. Oppmøte Larvik 16:30.'},
 // ... en rad per turdag
];
```

`todayRender()` finner dagens dato (`d` på ISO-form `ÅÅÅÅ-MM-DD`), og:
- **På en turdag:** viser etappe (`t`), plan (`plan`), «Husk» (`husk`) + dagens vær for
  `lat`/`lon`. «Dag X av N» – oppdater totalen (`' av 14'`) til antall turdager.
- **Før turen:** nedtelling. Sett start-datoen: `var start = new Date('2026-07-06T00:00');`
- **Etter turen:** fast «velkommen hjem»-tekst. Tilpass teksten.

## Vær (Open-Meteo)

Ingen API-nøkkel. Stoppene i nedtrekkslista:

```javascript
var WX_STOPS = [
  {n:'Aalborg', lat:57.05, lon:9.92, from:'2026-07-07'},
  // from = datoen appen auto-velger dette stoppet fra
];
```

Henting (flere dagers varsel):
```javascript
var url = 'https://api.open-meteo.com/v1/forecast?latitude='+lat+'&longitude='+lon+
  '&daily=weather_code,temperature_2m_max,temperature_2m_min,precipitation_probability_max'+
  '&timezone=auto&forecast_days=7';
fetch(url).then(r => r.json()).then(function(d){ /* render d.daily */ });
```

`wxDesc(code)` oversetter WMO-værkoder til emoji + norsk tekst (0=klart, 3=skyet,
61-67=regn, 95+=torden osv.). `wxGeo()` bruker `navigator.geolocation` for «📍 Her».

## Delt økonomi (Supabase)

```javascript
var SB_URL = 'https://DITTPROSJEKT.supabase.co';
var SB_KEY = 'sb_publishable_...';          // PUBLISERBAR nøkkel – trygg i klient
var ECO_RATES = {NOK:1, EUR:11.70, DKK:1.57, GBP:13.50};  // kurs til NOK for summering
var ECO_CACHE = 'roadtrip2026_eco_cache';   // localStorage-nøkkel for offline

function ecoHeaders(extra){
  var h = {'apikey':SB_KEY, 'Authorization':'Bearer '+SB_KEY};
  if(extra){ for(var k in extra){ h[k]=extra[k]; } } return h;
}
```

REST-kall (PostgREST):
- **Hent:** `GET {SB_URL}/rest/v1/expenses?select=*&order=created_at.asc`
- **Legg til:** `POST {SB_URL}/rest/v1/expenses` med header `Prefer: return=representation`
  og body `{title, category, amount, currency}`
- **Slett:** `DELETE {SB_URL}/rest/v1/expenses?id=eq.<id>`

Feiler kallet: les fra `localStorage[ECO_CACHE]` og vis «⚠️ Offline». Ved vellykket
hent: skriv tilbake til cachen. Kolonnemapping i `ecoFetch()`:
`{id:r.id, desc:r.title, cat:r.category, amt:Number(r.amount), cur:r.currency}`.

Tabell `expenses`: `id` (bigint PK auto), `title` (text), `category` (text),
`amount` (numeric), `currency` (text), `created_at` (timestamptz default now()).
RLS på, med policyer som tillater select/insert/delete for `anon`.

## Valutakalkulator (Frankfurter)

```javascript
var CC_RATE = 11.70;  // fast fallback (NOK per EUR) når offline
function ccFetchRate(){
  fetch('https://api.frankfurter.dev/v1/latest?base=EUR&symbols=NOK')
    .then(r => r.json())
    .then(function(d){ if(d && d.rates && d.rates.NOK){ CC_RATE = d.rates.NOK; /* ... */ } })
    .catch(function(){ /* behold fallback, vis "fast kurs (offline)" */ });
}
```

Bytt `base`/`symbols` til valutaene som er relevante for reisemålet.

## Pakkeliste per person

Hver person er en `.pack-cat`-blokk; hver vare en `.pitem` med `data-person`:

```html
<div class="pack-cat">
  <!-- overskrift + fremdriftsbar med id=bar-janiche / stats-janiche -->
  <div class="pitems">
    <label class="pitem">
      <input type="checkbox" data-person="janiche" onchange="updatePersonProgress('janiche')">
      <span><span>Pass (gyldig)</span><span class="pnote">Sjekk utløpsdato!</span></span>
    </label>
    <!-- flere varer ... -->
  </div>
</div>
```

```javascript
function updatePersonProgress(person) {
  const all = document.querySelectorAll(`input[data-person="${person}"]`);
  const checked = document.querySelectorAll(`input[data-person="${person}"]:checked`);
  const pct = all.length ? Math.round(checked.length/all.length*100) : 0;
  document.getElementById('bar-'+person).style.width = pct + '%';
  document.getElementById('stats-'+person).textContent =
    `${checked.length} av ${all.length} pakket (${pct}%)`;
  all.forEach(cb => cb.closest('.pitem').classList.toggle('checked', cb.checked));
}
```

Merk: avkryssing lagres ikke mellom besøk i originalen – den viser bare fremdrift i
økten. Vil Janiche at haker skal huskes, lagre hver `data-person`+indeks i
`localStorage` i `onchange` og les dem tilbake ved oppstart (samme mønster som
`ECO_CACHE`).

## Dagsplan-trekkspill

```javascript
function toggleDay(hd) {
  const body = hd.nextElementSibling;
  const chev = hd.querySelector('.dchev');
  const open = body.classList.toggle('open');
  chev.style.transform = open ? 'rotate(180deg)' : '';
}
```

HTML: en klikkbar `.dhead` (med `onclick="toggleDay(this)"` og en `.dchev`-pil) fulgt
av en `.dbody` som får klassen `open` når den utvides.

## Initialisering nederst

Nederst i `<script>` kjøres oppstarten – oppdater personlista så den matcher
pakkelista:

```javascript
['janiche','jon','emma','martine','felles'].forEach(p => updatePersonProgress(p));
ecoFetch();        // hent delt økonomi
ccRateLabel(null); // vis fallback-kurs
ccFetchRate();     // oppdater til live kurs
wxInit();          // fyll vær-nedtrekksliste
todayRender();     // tegn "i dag"-kortet
```
