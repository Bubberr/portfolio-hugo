---
title: "Julemarked på Engestofte Gods — Del 2: Admin-panel, CSV-export og deploy"
categories: ["Projekt", "Fullstack"]
tags: ["nextjs", "supabase", "tailwind", "fullstack", "admin", "vercel"]
date: 2026-05-27
draft: false
showauthor: true
authors:
  - Bubberr
---

## Hvor vi slap

I [del 1](/posts/julemarked-del-1/) byggede jeg tilmeldingsformularen: visuel standtype-vælger, realtidsprisberegning og en API route der gemmer ansøgninger i Supabase. Nu manglede den anden halvdel — det interface Lise Egeskov bruger til at behandle ansøgningerne.

---

## Admin-panelet

Admin-siden er passwordbeskyttet. Det er ikke OAuth eller Supabase Auth — det er et enkelt felt der tjekker mod `ADMIN_PASSWORD` i miljøvariablerne. Det er tilstrækkeligt for dette use case: én person med adgang, ingen offentlig login-side, ingen brugeradministration.

Når password er korrekt gemmes en session-cookie, og brugerens tilstand holdes lokalt i React-state. Logikken i `/api/admin/route.js` håndterer både autentificering og datahentning:

```js
// GET: hent alle bookings (kræver korrekt password i header)
export async function GET(request) {
  const adgangskode = request.headers.get('x-admin-password')

  if (adgangskode !== process.env.ADMIN_PASSWORD) {
    return Response.json({ fejl: 'Forkert adgangskode' }, { status: 401 })
  }

  const { data, error } = await supabase
    .from('bookings')
    .select('*')
    .order('created_at', { ascending: false })

  if (error) return Response.json({ fejl: error.message }, { status: 500 })
  return Response.json(data)
}
```

`PATCH`-endpointet håndterer statusopdateringer og tildeling af pladser:

```js
// PATCH: opdater status eller tildelt_plads
export async function PATCH(request) {
  const adgangskode = request.headers.get('x-admin-password')
  if (adgangskode !== process.env.ADMIN_PASSWORD) {
    return Response.json({ fejl: 'Forkert adgangskode' }, { status: 401 })
  }

  const { id, ...opdatering } = await request.json()

  const { error } = await supabase
    .from('bookings')
    .update(opdatering)
    .eq('id', id)

  if (error) return Response.json({ fejl: error.message }, { status: 500 })
  return Response.json({ besked: 'Opdateret' })
}
```

---

## Tre faneblade

Admin-panelet er organiseret i tre faner der svarer til de tre mulige statusser i databasen.

### Afventer

Den vigtigste fane i daglig brug. Her lander alle nye ansøgninger. Hver række viser:

- Virksomhedsnavn, kontaktperson, telefon, email
- Standtype + pris
- Om det er en ny eller erfaren stadeholder
- Kort beskrivelse

To knapper per række: **Godkend** og **Afvis**.

Godkend åbner en modal hvor Lise taster den tildelte plads (f.eks. "B12" eller "Laden nord"). Plads og status opdateres i ét PATCH-kald. Det er bevidst gjort som ét trin — plads og godkendelse hører sammen, og det giver ikke mening at godkende uden at tildele.

### Godkendte

Fuld tabel med alle godkendte bookings: samtlige felter, tildelt plads og beregnet totalpris. Pladsen kan redigeres inline — klik, rediger, tryk Enter eller klik væk.

Den beregnede totalpris genskaber `beregnPris()`-logikken fra formularen, men nu læst fra databaseværdierne:

```js
function totalpris(booking) {
  const stand = STAND_TYPER[booking.stand_type]
  return stand.pris + booking.antal_borde * 155 + booking.antal_stole * 45
}
```

En enkelt linje, men det er vigtigt at prisen beregnes ved visning fremfor at gemmes i databasen. Hvis priserne ændrer sig (f.eks. borde til 165 kr.), kan historiske bookings stadig vises med den pris der gjaldt da de blev oprettet — men det kræver at priserne på et tidspunkt versioneres. I denne version er priserne hardcodet, og det er godt nok til et enkelt julemarked.

### Afviste

En liste over afviste ansøgninger med en **Genaktivér**-knap. Den sætter status tilbage til `afventer` og lader Lise behandle ansøgningen igen — f.eks. hvis en stadeholder ringer og forklarer sig.

---

## CSV-export

En af de praktiske ting ved admin-panelet er eksport til CSV. Knappen genererer en fil med alle godkendte bookings og sender den til browseren:

```js
function eksporterCSV(bookings) {
  const kolonner = [
    'Virksomhed', 'Kontaktperson', 'Telefon', 'Email',
    'Standtype', 'Pris', 'Borde', 'Stole', 'Tildelt plads', 'Total'
  ]

  const rækker = bookings.map(b => [
    b.virksomhedsnavn,
    b.kontaktperson,
    b.telefon,
    b.email,
    b.stand_type,
    STAND_TYPER[b.stand_type]?.pris ?? '',
    b.antal_borde,
    b.antal_stole,
    b.tildelt_plads ?? '',
    totalpris(b)
  ])

  const csvIndhold = [kolonner, ...rækker]
    .map(r => r.map(v => `"${String(v).replace(/"/g, '""')}"`).join(','))
    .join('\n')

  const blob = new Blob(['﻿' + csvIndhold], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'godkendte-stadeholdere.csv'
  a.click()
}
```

To detaljer der sparer tid:

**BOM-markøren (`﻿`)** — uden den åbner Excel danske tegn (æ, ø, å) forkert på Windows. BOM'en fortæller Excel at filen er UTF-8.

**Anførselstegn-escaping** — alle felter pakkes i anførselstegn og interne `"` fordobles. Det sikrer at adresser, beskrivelser og firmanavne med kommaer ikke ødelægger kolonnestrukturen.

---

## Mappestruktur

```
app/
  page.jsx                  ← offentlig tilmeldingsformular
  admin/
    page.jsx                ← admin-overblik (passwordbeskyttet)
  api/
    bookings/
      route.js              ← POST: gem ny booking
    admin/
      route.js              ← GET: hent alle, PATCH: opdater status/plads
lib/
  supabase.js               ← Supabase-klient
  standtyper.js             ← STAND_TYPER konstanter + beregnPris()
```

`standtyper.js` er den vigtigste delte fil — den importeres af formularen (for at vise valgkort og beregne pris), af admin-tabellen (for at vise prisinformation) og af CSV-eksporten. Det er den eneste kilde til sandhed om hvad en standtype koster og hedder.

---

## Deploy til Vercel

```bash
npm install -g vercel
vercel
```

Vercel genkender Next.js-projekter automatisk. Det eneste der kræves manuelt er at sætte miljøvariablerne i Vercel-dashboardet:

| Variabel | Beskrivelse |
|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase projekt-URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase anon-nøgle |
| `ADMIN_PASSWORD` | Adgangskode til admin-siden |

`NEXT_PUBLIC_`-præfikset betyder at variablen bundtes med klient-koden — det er nødvendigt for Supabase-klienten der bruges i browser-kontekst. `ADMIN_PASSWORD` har ikke præfikset og er kun tilgængelig server-side.

---

## Hvad jeg lærte

**Supabase er hurtig at komme i gang med.** Database, hosted API og en JavaScript-klient klar til brug i én platform. Jeg brugte mere tid på formularen end på databaseintegration — det er den rigtige rækkefølge.

**Next.js App Router er fornuftig, men kræver lidt omskoling.** Forskellen på Server Components og Client Components er ikke altid intuitiv. Formularen er en Client Component fordi den håndterer state. API routes er server-only. Det virker, men fejlbeskederne når man blander dem er ikke altid hjælpsomme.

**Reelle projekter stiller andre krav.** Da jeg startede vidste jeg at det var til et specifikt arrangement med en specifik person som bruger. Det betød at jeg tænkte anderledes over brugeroplevelsen end jeg gør i skoleeksempler — hvad sker der hvis Lise godkender en booking og bagefter opdager at hun tastet forkert plads? Det er derfor "Godkendte" har inline-redigering af tildelt plads.

**CSV-export er undervurderet.** Lise arbejder i Excel, ikke i databaser. Det er ikke en begrænsning — det er kontekst. Et system der ikke kan give data ud i et format brugeren allerede kender, er et system der kun halvt løser problemet.

---

## Status

Systemet er live og tilgængeligt:

- **Tilmeldingsformular:** [engestoftegods.vercel.app](https://engestoftegods.vercel.app/)
- **Admin-panel:** [engestoftegods.vercel.app/admin](https://engestoftegods.vercel.app/admin)

Tilmeldingsformularen er offentlig. Admin-panelet kræver adgangskoden der er sat i Vercel-miljøvariablerne.

---

## Kontekst: fire iterationer, fire gruppemedlemmer

Dette projekt er del af et gruppeforløb hvor hvert medlem er ansvarlig for én iteration af systemet. Planen er fire iterationer i alt — én per person. Denne første iteration etablerer grundlaget: database, offentlig formular og admin-panel med fuld CRUD-funktionalitet.

De kommende iterationer kan bygge videre herfra — f.eks. automatisk bekræftelsesmail ved godkendelse, et kortudsnit over standplaceringer eller en offentlig oversigt over tilmeldte stadeholdere til Engestoftes hjemmeside.
