# Boggel – Architecture

## Tech stack

Den tekniska utgångspunkten är kodbasen som exporterades från Lovable via GitHub (React/Tailwind + Supabase-konfiguration). Claude Code bygger vidare på den tekniskt. Den visuella designen kommer däremot inte längre från Lovable-exporten, utan från två godkända prototyper framtagna i Claude Design, som ligger i design-reference/. Se Frontend nedan och design-reference/HANDOFF.md.

## Frontend
React/Tailwind, i den befintliga kodbasen.

Visuellt facit är prototyperna i design-reference/, inte Lovable-exporten.

design-reference/kompass/prototyp.html: Kompass-läget, koncept "Kompassen lyssnar"
design-reference/skarp/prototyp.html: Skärp-läget, koncept "Kartan"

Läs design-reference/HANDOFF.md innan du rör frontend. Där står hur prototyperna körs, vad koncepten innebär och i vilken ordning arbetet ska göras.

Prototyperna är facit för layout, typografi, färger, animationer, övergångar och interaktioner. Designen är medvetet framtagen för att kännas varm och genomtänkt och för att hålla kvar användaren genom hela vägledningen.

Återskapa prototyperna troget. Förbättra eller gör inte om designen på eget initiativ, även om en "bättre" lösning känns tillgänglig.
Befintliga komponenter och stilar från Lovable-exporten får återanvändas tekniskt där de passar, men deras utseende ska följa prototyperna. Där Lovable-stilen och prototyperna skiljer sig åt gäller prototyperna.
Bygg det gemensamma (typografi, färgtokens, understrykningar, romber, stegindikator, rörelser) som ett delat designsystem som båda lägena använder, översatt till Tailwind-konfiguration och återanvändbara komponenter.
Prototyperna gäller för design och upplevelse, men flödets steg och avslutningsmallen styrs av CLAUDE.md. Prototyperna visar ett förkortat flöde med påhittat innehåll.
Nya vyer eller steg som inte finns i prototyperna ska byggas i samma visuella språk.
Om något i prototyperna inte går att bygga som det är, eller behöver ändras av tekniska skäl (t.ex. prestanda, tillgänglighet): fråga eller flagga det innan det görs, istället för att bara avvika.
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
