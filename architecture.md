# Boggel – Architecture

## Tech stack

Utgångspunkten är draften byggd i Lovable (flöde + färdig visuell design), exporterad via Lovables GitHub-integration till en riktig, redigerbar kodbas (React/Tailwind + Supabase-konfiguration). Claude Code bygger vidare på denna kodbas — den återskapar inte gränssnittet från beskrivning.

## Frontend

React/Tailwind, från Lovable-exporten.

Den visuella designen (färger, former, layout) är färdig och medvetet framtagen för att kännas snygg, välkomnande och för att få användare att stanna kvar genom hela vägledningen. Den befintliga Lovable-kodbasen är facit för visuell stil.

- Bygg vidare på och återanvänd befintliga komponenter, färger och stilar.
- Föreslå eller gör inte om designen från grunden, även om en "bättre" lösning känns tillgänglig.
- Det är okej att förbättra eller utöka styling när nya vyer/steg behöver det (t.ex. Kompass-läget, nya sektioner i backloggen) — men i samma visuella språk som redan finns.
- Om något i den visuella implementationen behöver ändras av tekniska skäl (t.ex. prestanda, tillgänglighet): fråga eller flagga det innan det görs, istället för att bara skriva över det.

## Backend

Två separata AI-roller, inte en enda AI:

- **Research-lager** (Perplexity el. motsvarande): hämtar live marknadsdata, konkurrenter, branschsignaler vid behov.
- **Kärn-AI** (Claude API): tar emot användarens input + researchresultat + systemprompt + ev. kunskapsbas-utdrag, gör själva resonemanget och genererar frågor/förslag/slutbeskrivning.

Claude Code bygger applikationskoden runt Claude API:et (prompthantering, orkestrering av anrop, strukturering av svar) — det handlar om att koda ett smart applikationslager, inte om att träna en egen modell.

Vikt läggs på **samtalsdesign i systemprompten** (följdfrågor, reflektion av mönster användaren inte själv uttalat, undvik generiska svar) snarare än en fristående "psykologikunskapsbas" — det är interaktionsdesignen, inte en dokumentsamling, som skapar upplevelsen av att verkligen bli förstådd.

## Database

Supabase, konfigurerad via Lovable-exporten.

## API structure

- Abstraktionslager mot researchleverantören (inte hårdkodat mot Perplexity specifikt), så att leverantör kan bytas ut.
- Cachning av återanvändbar research över användare (t.ex. 24–48h) för breda ämnen, istället för att fråga på nytt varje session — annars blir kostnaden per session onödigt hög.
- Claude API-anrop hanterar prompthantering, orkestrering och strukturering av svar i kärnflödet.
