---
title: "Julemarked på Engestofte Gods — Del 1: Spec, database og tilmeldingsformular"
categories: ["Projekt", "Fullstack"]
tags: ["nextjs", "supabase", "tailwind", "fullstack", "dansk", "database"]
date: 2026-05-20
draft: false
showauthor: true
authors:
  - Bubberr
---

## Projektet

Engestofte Gods afholder julemarked den 5.-6. december 2026. Tidligere foregik tilmeldingen via email og et PDF-skema — svar kom manuelt, og overblikket over hvem der havde ansøgt, hvad der var godkendt og hvem der manglede svar, lå spredt i en indbakke.

Opgaven: byg et system der samler hele processen. Stadeholdere udfylder en formular online. Lise Egeskov ser alle ansøgninger i et admin-panel, godkender eller afviser dem og tildeler pladser. Det hele bor i en database.

Det er et reelt brugssystem — ikke et øvelsesprojekt — og det stiller andre krav til kvalitet og brugervenlighed end det man typisk laver i skolesammenhæng.

---

## Tech stack

| Lag | Valg | Begrundelse |
|---|---|---|
| Frontend/backend | Next.js 14 (App Router) | Én repo, API routes og UI lever samme sted |
| Database | Supabase (PostgreSQL) | Hosted Postgres med autentificering, realtime og REST ud af boksen |
| Styling | Tailwind CSS | Hurtig at arbejde med, intet ekstra build-trin |
| Deploy | Vercel | Direkte integration med Next.js, ingen konfiguration |

Next.js App Router er stadig relativt nyt, men det passer godt til dette projekt: tilmeldingsformularen er en server-rendered side, admin-panelet bruger API routes, og alt bygges og deployes som én enhed.

---

## Database-design

Det første jeg lavede var databasetabellen. Den beskriver en booking fra end til end — fra kontaktoplysninger til admin-status:

```sql
create table bookings (
  id uuid default gen_random_uuid() primary key,
  created_at timestamp with time zone default now(),

  -- Virksomhedsinfo
  virksomhedsnavn text not null,
  kontaktperson   text not null,
  cvr             text,
  adresse         text,
  postnummer_by   text,
  telefon         text not null,
  email           text not null,
  website         text,

  -- Standvalg
  stand_type      text not null,
  antal_borde     integer default 0,
  antal_stole     integer default 0,

  -- Beskrivelser
  kort_beskrivelse  text not null,
  produkt_beskrivelse text,
  er_ny_stadeholder boolean default true,

  -- Admin
  status        text default 'afventer',
  tildelt_plads text,
  noter         text
);
```

Et par designbeslutninger værd at nævne:

**`status` som enum-lignende tekst** — jeg brugte `text` med tre mulige værdier (`afventer`, `godkendt`, `afvist`) frem for en PostgreSQL `ENUM`. Det giver mere fleksibilitet hvis der senere skal tilføjes en status, og Supabase håndterer valideringen fint uden database-constraints i første omgang.

**`er_ny_stadeholder` som boolean** — det er et simpelt felt, men det driver logik på to steder: om produktbeskrivelsesfeltet vises i formularen, og om admin-tabellen markerer rækken som "ny".

**Ingen separat tabel til standtyper** — standtyperne (A–H) er defineret som konstanter i applikationslaget, ikke i databasen. De ændrer sig sjældent, og det holder databasen enkel.

---

## Supabase-klienten

```js
// lib/supabase.js
import { createClient } from '@supabase/supabase-js'

export const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY
)
```

En enkelt fil, eksporteret som singleton. Alle API routes og komponenter importerer herfra.

---

## Tilmeldingsformularen

Formularen er offentlig og tilgængelig uden login. Den er designet med en julemarked-æstetik: varm, mørk grøn baggrund (`#1a3a2a`), rødbrune accenter og guldfarvet tekst på overskrifter. Målet er at det skal føles som Engestofte — hyggeligt og professionelt på samme tid.

Felterne følger det eksisterende ansøgningsskema i fast rækkefølge:

```
Virksomhedsnavn (påkrævet)
Kontaktperson (påkrævet)
CVR-nummer (valgfrit)
Adresse (valgfrit)
Postnummer og by (valgfrit)
Telefon/mobil (påkrævet)
Email (påkrævet)
Website (valgfrit)
Standtype (visuel vælger)
Antal borde / stole
Ny stadeholder? (toggle)
Kort beskrivelse
Produktbeskrivelse (kun hvis ny stadeholder)
```

Det interessante er standtype-vælgeren og prisberegningen — begge dele krævede lidt mere arbejde end resten af formularen.

---

## Standtype-vælgeren

Der er otte standtyper (A–H) med forskellig størrelse, lokation og pris. En simpel `<select>` ville fungere, men det ville kræve at brugeren selv husker forskellen på "Kostalden langside" og "Kostalden center". Jeg lavede i stedet et gitter af valgkort:

```js
const STAND_TYPER = {
  A: {
    navn: 'Udendørsstand',
    størrelse: '3×4 m',
    lokation: 'Udendørs',
    elektricitet: false,
    pris: 985,
    beskrivelse: 'Ingen elektricitet. Kan tilkøbes via Lise.',
  },
  B: {
    navn: 'Indendørs langside',
    størrelse: '3×3 m',
    lokation: 'Kostalden',
    elektricitet: true,
    pris: 1685,
    beskrivelse: '1 side mod publikum. Elektricitet 220V.',
  },
  // ... C–H
}
```

Hvert kort viser type-bogstavet, lokation, størrelse, om der er elektricitet og prisen. Det valgte kort fremhæves med en guldfarvet ramme. Det tager tre sekunder at sammenligne standtyper frem for at læse en prisliste.

---

## Prisberegning i realtid

Under formularen vises et dynamisk prissammendrag der opdateres mens brugeren udfylder:

```js
function beregnPris(standType, antalBorde, antalStole) {
  const stand = STAND_TYPER[standType]
  if (!stand) return null

  const standPris = stand.pris
  const bordePris = antalBorde * 155
  const stolePris = antalStole * 45
  const total = standPris + bordePris + stolePris

  return { standPris, bordePris, stolePris, total }
}
```

Priserne er ekskl. moms — det er standpraksis i B2B-sammenhæng, og det stemmer overens med Engestoftes skema.

```
Standleje (B – Kostalden langside):  1.685 kr + moms
Borde (2 stk):                         310 kr + moms
Stole (4 stk):                         180 kr + moms
─────────────────────────────────────────────────────
Total ekskl. moms:                   2.175 kr
```

---

## Indsendelse og fejlhåndtering

Formularen POST'er til `/api/bookings/route.js`, som validerer de påkrævede felter og gemmer til Supabase:

```js
// app/api/bookings/route.js
export async function POST(request) {
  const data = await request.json()

  const påkrævede = ['virksomhedsnavn', 'kontaktperson', 'telefon', 'email', 'stand_type', 'kort_beskrivelse']
  for (const felt of påkrævede) {
    if (!data[felt]) {
      return Response.json({ fejl: `Feltet "${felt}" er påkrævet` }, { status: 400 })
    }
  }

  const { error } = await supabase.from('bookings').insert([data])

  if (error) {
    return Response.json({ fejl: 'Noget gik galt. Prøv igen.' }, { status: 500 })
  }

  return Response.json({ besked: 'Ansøgning modtaget' }, { status: 201 })
}
```

Alle fejlbeskeder er på dansk og sigte mod at være handlingsanvisende — ikke "Internal Server Error", men "Noget gik galt. Prøv igen."

Ved succes vises:

> *Tak for din ansøgning! Du hører fra Lise Egeskov snarest. Husk at sende produktfotos til le@engestofte.dk hvis du er ny stadeholder.*

---

## Hvad kom del 1 frem til

Den offentlige side er klar: formularen tager imod ansøgninger, gemmer dem i Supabase og bekræfter til brugeren. Designet er tilpasset Engestoftes æstetik og fungerer på mobil og desktop.

Det der mangler er admin-siden: hvordan Lise ser ansøgningerne, godkender dem, tildeler pladser og eksporterer data til regneark. Det er emnet for del 2.
