# Boggel – Decisions

Viktiga beslut som redan är fattade och bekräftade — utgångspunkter för fortsatt arbete, inte öppna frågor.

## Visuell design bevaras, byggs inte om

Prototyperna i `design-reference/` (se `design-reference/HANDOFF.md`) är facit för visuell stil, inte textbeskrivningar av produkten och inte den ursprungliga Lovable-exporten. Designen är ett medvetet beslut framtaget för att kännas snygg och välkomnande, inte en placeholder. Styling kan förbättras/utökas för nya vyer, men i samma visuella språk som prototyperna. Tekniska ändringar av visuell implementation ska flaggas/diskuteras innan de görs.

## Styrt flöde, inte fri chatt

Produkten positioneras medvetet som "styrd metod framför fri chatt". Menyval styr flödet genom hela vägledningen, med fritext som override — inte en öppen chattupplevelse.

## Fast avslutningsmall

Slutresultatet av varje session följer alltid samma sex-punktsstruktur (idé, beskrivning, passform, särskiljbarhet, svaga punkter, nästa steg) och innehåller aldrig påhittade siffror eller namn.

## Två separata AI-roller

Research-lager (marknadsdata) och kärn-AI (resonemang, Claude API) hålls isär som två roller, inte en enda AI som gör allt.

## Research-arkitektur: leverantörsoberoende + cachning

Abstraktionslager mot researchleverantören byggs istället för att hårdkoda mot Perplexity specifikt. Återanvändbar research cachas över användare (24–48h) för breda ämnen, för att hålla kostnaden per session nere.

## Samtalsdesign prioriteras över kunskapsbas

Upplevelsen av att bli förstådd skapas genom interaktionsdesign i systemprompten (följdfrågor, reflektion av mönster), inte genom en fristående "psykologikunskapsbas".

## Affärsmodell differentierad per målgrupp

- Rådgivare/konsulter: licens per rådgivare, löpande.
- Enskilda grundare: betalning per idé/session, inte prenumeration — motiverat av att många grundare itererar genom flera idéer innan de landar rätt.
- Befintliga företag (produktfas 4): prenumeration, motiverad av kontinuerligt återkommande behov.

## Vad som medvetet inte byggs in i kärnflödet just nu

- Automatisk dokumentation / delad vy mellan flera möten för rådgivare — hör till en annan typ av produkt (klienthantering), hålls som roadmap-notering.
- Full sektionsindelad dashboard-komplexitet för alla användare — hör till produktfas 4, inte MVP:t.

## Kodbas-utgångspunkt

Lovable-draften exporteras via GitHub-integration innan Claude Code börjar bygga, så att en riktig, redigerbar kodbas (React/Tailwind + Supabase) finns att utgå från — istället för att gränssnittet återskapas från beskrivning.
