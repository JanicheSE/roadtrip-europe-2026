# 📌 Slik oppdaterer jeg reiseappen

**Appen live:** https://roadtrip-europe-2026.netlify.app

---

## ✅ Dette endrer jeg selv – rett i appen
Lagres med en gang, ingen hjelp nødvendig:
- **Økonomi** – legge til / slette utgifter (deles med alle · trykk 🔄 for nyeste)
- **Pakkeliste** – krysse av det som er pakket
- **Vær** – oppdaterer seg selv

## 🛠️ Alt annet – bare be Claude Code
Åpne **Claude Code** og skriv på vanlig norsk. Ingen koding.

**Eksempler jeg kan skrive:**
- «Legg inn hotellet vi booket i Amsterdam: [navn + bekreftelsesnr]»
- «Bytt Hamburg-hotellet fra forslag til booket»
- «Endre dag 7 til å starte kl 10»
- «Legg inn Martines togbillett Birmingham→Gatwick: [detaljer]»
- «Legg til et stopp / en aktivitet …»

## 🔁 Flyten
```
Jeg sier hva jeg vil endre
        ↓
Claude redigerer + pusher til GitHub
        ↓
Netlify publiserer automatisk (~30 sek)
        ↓
Live for alle – samme lenke, samme ikon
```

---

## 🧰 Fakta (til Claude om nødvendig)
- **Kode:** `C:\Users\aeg81\Documents\GitHub\roadtrip-europe-2026\index.html` (én fil)
- **GitHub:** `JanicheSE/roadtrip-europe-2026` (gren `main`)
- **Netlify:** prosjekt `roadtrip-europe-2026` – auto-deploy ved push
- **Delt økonomi:** Supabase (ligger allerede inne)
- **Mal for nye reiser:** `C:\Users\aeg81\Claude\Templates\reiseapp-mal\`

💡 Tips: du kan be om oppdateringer **når som helst** – før turen eller midt i ferien fra mobilen via Claude Code.
