# Boggel – Decisions

Viktiga beslut som redan är fattade och bekräftade — utgångspunkter för fortsatt arbete, inte öppna frågor.

## Visuell design bevaras, byggs inte om

Prototyperna i `design-reference/` (se `design-reference/HANDOFF.md`) är facit för visuell stil, inte textbeskrivningar av produkten och inte den ursprungliga Lovable-exporten. Designen är ett medvetet beslut framtaget för att kännas snygg och välkomnande, inte en placeholder. Styling kan förbättras/utökas för nya vyer, men i samma visuella språk som prototyperna. Tekniska ändringar av visuell implementation ska flaggas/diskuteras innan de görs.

## Styrt flöde, inte fri chatt

Produkten positioneras medvetet som "styrd metod framför fri chatt" — en öppen chattupplevelse är det INTE, i något läge.

I Kompass-läget betyder det fortfarande menyval med fritext som override, som tidigare.

I Skärp-läget betyder det sedan flödesombyggnaden i stället: användaren skriver fritt FÖRST i de steg där det spelar roll ("Du först"), och AI-genererade förslag/alternativ visas efteråt eller via en uttrycklig flyktväg ("Jag vet inte — visa förslag") — fortfarande ett fast, stegvis flöde genom tio namngivna steg, bara med en annan turordning mellan AI och användare än Kompass. Se `docs/skarp-flode.md`, princip 2.

## Fast avslutningsmall (Kompass-läget)

Kompass-lägets slutresultat följer alltid samma sex-punktsstruktur (idé, beskrivning, passform, särskiljbarhet, svaga punkter, nästa steg) och innehåller aldrig påhittade siffror eller namn.

Skärp-läget har en EGEN avslutning sedan Skärp-flödet byggdes om: ett
"slutkort och test" (idé i en mening, problemram, kunden i situationen,
mekanism, det unika, affärsgrund, en sexfaktorig idéstyrka-profil utan
totalsumma, svagaste antagandet och stoppregeln, ett testkort, tre
frågor till rådgivaren). Samma princip om att aldrig hitta på siffror
eller namn gäller där också. Se `docs/skarp-flode.md`.

## Två separata AI-roller

Research-lager (marknadsdata) och kärn-AI (resonemang, leverantörsoberoende — se `docs/architecture.md`) hålls isär som två roller, inte en enda AI som gör allt.

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
