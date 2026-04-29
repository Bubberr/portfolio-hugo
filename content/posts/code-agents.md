---
title: Mine første oplevelser med kodeagenter
categories: ["AI", "Tools"]
tags: ["ai", "agents", "claude", "developer-tools", "workflow"]
date: 2026-04-29
draft: false
showauthor: true
authors:
  - Bubberr
---

## Fra autocomplete til agentic AI

Jeg har brugt GitHub Copilot en del – den slags AI der fuldfører en linje eller foreslår en funktion. Det er nyttigt, men det er stadig mig der navigerer, klipper, søger og fikser. Kodeagenter er noget andet.

En kodeagent læser filer, søger i kodebasen, retter fejl og kører kommandoer  i ét hug, mens jeg beskriver hvad jeg vil have. Det var den oplevelse der overraskede mig mest, da jeg begyndte at bruge **Claude Code**.

---

## Hvad er en kodeagent?

En kodeagent er et AI-system der ikke bare genererer tekst, men *handler* i et miljø. Den har adgang til værktøjer:

| Værktøj | Hvad agenten gør |
|---|---|
| Filsystem | Læser og skriver filer |
| Terminal | Kører kommandoer og tests |
| Søgning | Finder relevante steder i kodebasen |
| Web | Slår dokumentation op |

Det betyder at agenten kan tage en opgave som "tilføj et endpoint der returnerer rubrikken som JSON" og selv finde ud af: *hvilken fil tilhører det? Hvad er konventionen i projektet? Hvad mangler der?* – og så bare gøre det.

---


## Hvad overraskede mig

**Den spørger, når det er uklart.** Jeg forventede at agenten bare ville gætte og producere noget forkert. Men hvis opgaven er tvetydig, stopper den og afklarer – lidt som en kollega der ikke vil spilde tid på at løse det forkerte problem.

**Den ser på hele projektet.** Copilot ser hvad der er åbent i editoren. Claude Code søger aktivt i hele kodebasen, tjekker afhængigheder og følger konventionerne den finder. Det giver en anden kvalitet af forslag.

**Den er ikke altid rigtig.** Agenten laver fejl, ligesom alle andre. Den kan misforstå konteksten eller lave antagelser der ikke holder. Det er stadig mit ansvar at reviewe det den laver – jeg bare gør det hurtigere end jeg skriver fra bunden.

---

## Ændringer i workflow

Før agenter brugte jeg typisk denne loop:

```
skriv kode → fejl → google → Stack Overflow → prøv igen
```

Med en kodeagent ser loopen mere sådan ud:

```
beskriv problemet → review løsning → godkend eller korriger
```

Det er en markant forskel i hvad jeg bruger mentale ressourcer på. Mindre syntaks, mere arkitektur og designbeslutninger.

---

## Begrænsninger jeg har stødt på

- **Store refaktoreringer kræver præcise instruktioner.** Jo mere åben en opgave er, jo mere generisk bliver output.  
- **Agenten ved ikke hvad du *ikke* sagde.** Hvis du glemmer at nævne et krav, glemmer den det også.  
- **Kontekstvinduet er ikke uendeligt.** På store projekter kan agenten miste overblikket over filer den har set.

Det handler i høj grad om at lære at *prompte godt* – at give nok kontekst til at agenten kan tage gode beslutninger.

---

## Refleksion

Kodeagenter er ikke "AI der erstatter programmøren". De er et værktøj der forskyder arbejdet – fra *at skrive kode* til *at beskrive og reviewe kode*. Det kræver stadig at man forstår koden man godkender.

For mig har det gjort det nemmere at eksperimentere. Jeg tøver mindre med at prøve en ny tilgang, fordi iterationerne er hurtigere. Det har gjort projekter som [RAG-automatiseringen](/posts/rag-automated/) og LLM API'et hurtigere at bygge end de ville have været ellers.

Jeg er stadig i begyndelsen af at forstå, hvornår agenter hjælper og hvornår de er i vejen. Men den første oplevelse var overbevisende nok til at de nu er en fast del af min arbejdsgang.
