# Boggel – Decisions

Viktiga beslut som redan är fattade och bekräftade — utgångspunkter för fortsatt arbete, inte öppna frågor.

## Visuell design bevaras, byggs inte om

Prototyperna i `design-reference/` (se `design-reference/HANDOFF.md`) är facit för visuell stil, inte textbeskrivningar av produkten och inte den ursprungliga Lovable-exporten. Designen är ett medvetet beslut framtaget för att kännas snygg och välkomnande, inte en placeholder. Styling kan förbättras/utökas för nya vyer, men i samma visuella språk som prototyperna. Tekniska ändringar av visuell implementation ska flaggas/diskuteras innan de görs.

## Styrt flöde, inte fri chatt

Produkten positioneras medvetet som "styrd metod framför fri chatt" — en öppen chattupplevelse är det INTE, i något läge.

I Kompass-läget betyder det fortfarande menyval med fritext som override, som tidigare.

I Skärp-läget är det fortfarande ett fast, stegvis flöde genom tio namngivna steg, men sedan "Ge först, fråga sen" (se nedan och §0.14 i `docs/skarp-flode.md`) levererar Lindo ett eget bidrag efter varje svar och frågar sedan. Användaren skriver själv först bara i steg 1 (idén) och i steg 5 och 7 (egen problemram och kärna), med flyktvägen "Jag vet inte — visa förslag" där.

## Fast avslutningsmall (Kompass-läget)

Kompass-lägets slutresultat följer alltid samma sex-punktsstruktur (idé, beskrivning, passform, särskiljbarhet, svaga punkter, nästa steg) och innehåller aldrig påhittade siffror eller namn.

Skärp-läget har en EGEN avslutning sedan Skärp-flödet byggdes om: ett
slutkort utan sifferbetyg (fas 3): idén i en mening, det som är starkt
(med användarens egna ord) först, en eventuell insikt, en nivå per
område — "Starkt", "På god väg" eller "Behöver mer underlag" — med en
mening om nästa steg, idén del för del (idékartans nio fält och
affärsgrunden), "Saker värda att testa" med stoppregeln, ett första test
och tre frågor att ta med till rådgivaren. Samma princip om att aldrig
hitta på siffror eller namn gäller där också. Se §0 i
`docs/skarp-flode.md`.

## Två separata AI-roller

Research-lager (marknadsdata) och kärn-AI (resonemang, leverantörsoberoende — se `docs/architecture.md`) hålls isär som två roller, inte en enda AI som gör allt.

## Research-arkitektur: leverantörsoberoende + cachning

Abstraktionslager mot researchleverantören byggs istället för att hårdkoda mot Perplexity specifikt. Återanvändbar research cachas över användare (24–48h) för breda ämnen, för att hålla kostnaden per session nere.

## Samtalsdesign prioriteras över kunskapsbas

Upplevelsen av att bli förstådd skapas genom interaktionsdesign i systemprompten (följdfrågor, reflektion av mönster), inte genom en fristående "psykologikunskapsbas".

## Skärp: ge först, fråga sen (ersätter "tankemotor, inte idémaskin")

Ett riktigt test visade att verktyget (då kallat Boggel, nu Lindo) var för passivt: det frågade men
tillförde nästan inget eget, fastnade i planens detaljer (vem som ska
vara testperson) och vissa frågor var otydliga. Orsaken var
grundregeln — "tankemotor, inte idémaskin", "AI:n frågar, speglar,
strukturerar och utmanar", "skriver aldrig färdiga idéer", "användaren
tänker först" — som gjorde att all kunskap bara blev frågor. Beslut:

- **Ge först, fråga sen.** Efter varje svar levererar Lindo 2–4 korta
  punkter (perspektiv, konkreta förslag, möjligheter, risker) i samma
  anrop som redan bedömer svaret — inga nya AI-anrop. Bidraget är
  huvudinnehållet; frågan är sekundär och låser upp nästa bidrag.
- **Idén, inte planen.** Frågor och bidrag handlar om problem, kund,
  värde, skillnad mot alternativen och affär. Genomförandets logistik
  (vem som testar, när, rekrytering) hör bara hemma i slutkortets
  testkort; en mjuk kontroll stoppar den i frågor och bidrag.
- **Användaren först bara där det räknas:** steg 1 (idén) och steg 5
  och 7 (egen problemram och kärna).
- **Ärligheten står kvar:** bidragen märks "Lindos förslag", påstår
  inga fakta utöver användarens svar och researchen, och insikterna
  citerar fortfarande användarens ord. Steg 6:s positioneringar får bara
  använda konkurrenter och fakta ur researchen.
- Den gamla principen ska inte återinföras i prompter, roller eller
  dokument.

## Skärp: verktyget heter Lindo, rådet ersätter Skeptikern, ingen sidscroll

Beslut (§0.15 i `docs/skarp-flode.md`):

- **Lindo är verktyget, Boggel är företaget.** Gränssnitt, prompter och
  dokument säger Lindo ("Lindos förslag", "Lindos tolkning", "Lindos
  råd"). Lindo pratar i jag-form, har egna åsikter, minns det
  användaren sagt och är varm, nyfiken, rak och lite lekfull — men
  låtsas aldrig vara en människa.
- **Rätt nivå och vardagligt språk.** Frågor och förslag gäller
  målgruppen och den typiska kunden, aldrig enskilda personer; inga
  facktermer om användaren inte själv använt dem (mjuk kontroll
  `jargonProblem`).
- **Lindos råd ersätter Skeptikern** — fem röster som får säga emot
  varandra och en Ordförande med beslut, största risk och första steg,
  i ett eget anrop (ingen ökning av antalet anrop). Rådet ersätter
  "Saker värda att testa" och föder slutkortets testkort.
- **Ingen klippt text, ingen sidscroll.** Steget får plats på 1366×768,
  1440×900 och mobil; listor och idékartan scrollar internt.
- **Tillbaka utan dubbelarbete.** Varje visad skärm sparas med allt den
  byggde på; oförändrat framåt spelar upp det sparade (inga nya
  AI-anrop, ingen dubbel sparning eller kartflytt), ett ändrat svar kör
  om de påverkade stegen. Research följer den befintliga regeln, och
  slutkortet sparas en gång per session.

## Skärpan: deterministiskt verktygsval, heuristiker och oberoende mätning

Skärp-läget har sedan Skärpan en inbyggd kunskapsbas av tankeverktyg
och kända fallgropar per idétyp (§0.13 i `docs/skarp-flode.md`).
Besluten bakom:

- **Deterministiskt urval i stället för ett extra AI-anrop.** Vilka
  verktyg ett steg får välja bland avgörs av vanlig kod (`selectTools`):
  samma indata ger samma verktyg, urvalet går att enhetstesta och kostar
  ingenting, och eval och flödestest kan jämföra körningar rakt av. Det
  enda nya AI-anropet är klassificeringen efter steg 1. Modellen väljer
  sedan ETT av de 2–3 erbjudna verktygen och anger det i `lens`, vilket
  kontrolleras i kod. Taket på omkring 25 logiska anrop per session
  står fast (18 utan följdfrågor).
- **Heuristiker, aldrig statistik.** Kunskapsbasen innehåller bara
  tumregler och perspektivskiften — inga andelar, frekvenser, procent
  eller "studier visar" — eftersom Lindo inte har underlag för sådana
  påståenden. Blocket säger "heuristiker, inte fakta", rösten förbjuder
  hänvisningar till egen erfarenhet och till hur ofta något händer, och
  en mjuk kontroll stoppar statistiska påståenden.
- **Sagt är inte belagt.** Att något står i idén betyder inte att det
  är bevisat. Klassificeringen skiljer på `stated` (fråga inte VAD det
  är) och `open` (pröva hur säkert det är), och samma område kan vara
  båda; bevis räknas bara som sagt när det är beteende (betalning,
  återkommande användning, egna siffror), aldrig åsikter eller planer.
  Lindo ställer därför inte om grundfrågan, men prövar det som redan
  sagts.
- **Oberoende mätning.** Kvaliteten bedöms av en egen bedömarmodell
  (standard `claude-opus-5-5`) med en egen prompt — inte Lindo och inte
  personsimulatorn — som aldrig ser etiketter, commit, version eller
  "före/efter"; i den parvisa jämförelsen lottas A och B per par.
  Inspelning och uppspelning (`--record`/`--replay`) gör att en
  förbättringsrunda bara ändrar det som faktiskt ändrats, bedömarbruset
  mäts en gång med `--regrade`, och en ändring behålls bara om
  förbättringen är minst max(0,25, 2 × brusets SD). En förbättring måste
  vara en allmän heuristik, aldrig något som bara passar evalpersonernas
  branscher. Se `docs/skarp-kvalitet.md`.
- **Huvudbok för budgeten.** Varje riktig evalkörning — även
  ombedömning och parvis jämförelse — skrivs i
  `eval-results/ledger.jsonl`. En körning startar inte om huvudboken
  plus prognosen överstiger totalbudgeten (standard 15 USD), en persona
  startas aldrig om den inte hinner bli klar inom körningens tak, och
  budgeten går före antalet förbättringsrundor.

Det ändrar inte beslutet ovan om samtalsdesign: kunskapsbasen är
tankeverktyg för frågorna, ingen psykologikunskapsbas, och den når
användaren bara genom samtalsdesignen — en fråga, en spegling, en
diskret perspektivetikett.

## Affärsmodell differentierad per målgrupp

- Rådgivare/konsulter: licens per rådgivare, löpande.
- Enskilda grundare: betalning per idé/session, inte prenumeration — motiverat av att många grundare itererar genom flera idéer innan de landar rätt.
- Befintliga företag (produktfas 4): prenumeration, motiverad av kontinuerligt återkommande behov.

## Vad som medvetet inte byggs in i kärnflödet just nu

- Automatisk dokumentation / delad vy mellan flera möten för rådgivare — hör till en annan typ av produkt (klienthantering), hålls som roadmap-notering.
- Full sektionsindelad dashboard-komplexitet för alla användare — hör till produktfas 4, inte MVP:t.

## Kodbas-utgångspunkt

Lovable-draften exporteras via GitHub-integration innan Claude Code börjar bygga, så att en riktig, redigerbar kodbas (React/Tailwind + Supabase) finns att utgå från — istället för att gränssnittet återskapas från beskrivning.
