# Boggel

## Project documentation

När informationen behövs, läs:

- `docs/product.md` – vad Boggel är och hur produkten fungerar
- `docs/architecture.md` – teknisk arkitektur
- `docs/roadmap.md` – planerade funktioner
- `docs/decisions.md` – viktiga tekniska beslut
- `docs/skarp-flode.md` – Skärp-lägets flöde, AI-roller, scheman och
  evidensmärkning. Källan för Skärp-läget — ersätter äldre beskrivningar
  av Skärp-flödet i de andra dokumenten ovan, som pekar hit där de
  tidigare beskrev det i detalj. Läs innan du rör Skärp-lägets kod eller
  prompter.
- `design-reference/HANDOFF.md` – designöverlämning och visuellt facit för frontend. Läs innan du gör något frontendarbete.

## Skärp-läget, kort

Verktyget heter **LINDO**; **Boggel är företaget**. I gränssnitt,
prompter och dokument heter AI:n Lindo ("Lindos förslag", "Lindos
tolkning", "Lindos råd", "Fråga Lindo om idén"). Ordet Boggel står bara
där det verkligen avser företaget (och i designsystemets namn,
`boggel-ui`). Lindo är en person men låtsas aldrig vara en människa:
jag-form, egna åsikter som sägs rakt ("Ärligt talat tror jag att …",
"Om jag var du skulle jag …"), minns vad användaren sagt och refererar
till det naturligt, varm, nyfiken, rak och lite lekfull — som en kompis
som kan affärer, över en kopp kaffe. Se §0.15 i `docs/skarp-flode.md`.

Skärp-läget är INTE ett styrt menyflöde längre (det var det gamla
flödet). Grundprincipen är GE FÖRST, FRÅGA SEN (§0.14): Lindo är en
expert som tänker MED användaren och tillför mer än användaren själv
gör. Efter varje svar levererar Lindo ett eget bidrag på 2–4 korta
punkter — perspektiv, konkreta förslag (en ny målgrupp, ett annat sätt
att ta betalt, ett sätt att sticka ut, en kil in på marknaden),
möjligheter och risker — märkt "Lindos förslag". Varje punkt är en kort
rubrik (högst sex ord) och EN mening, visad skannbart med Boggels
grafiska profil (gul accent, romber, papperskänsla, mjuk animation).
Bidraget är huvudinnehållet; frågan är kort, sekundär och handlar om
idén (problem, kund, värde, skillnad mot alternativen, affär), aldrig om
genomförandets logistik. Användaren skriver själv först BARA i steg 1
(idén) och i steg 5 och 7 (egen problemram och kärna) innan Lindos
förslag visas. Den gamla principen — "tankemotor, inte idémaskin",
"AI:n frågar, speglar, strukturerar och utmanar", "skriver aldrig
färdiga idéer", "Du först" i alla steg — gäller INTE längre och ska inte
återinföras.

**Nivåregeln:** frågor och förslag handlar om målgruppen och den typiska
kunden (ICP), aldrig om enskilda personer ("Tänk på en typisk grundare
med en luddig idé …", inte "Vilken av prospektets klienter …"); exempel
med namngivna personer används inte. **Språkregeln:** vardagligt och
tydligt, inga facktermer om användaren inte själv använt dem ("Vad är
det som gör att folk drar sig för att köpa?", inte "Vilken friktion
uppstår i köpprocessen?"); `jargonProblem` stoppar dem.

Flödet har tio steg (Din idé → Varför du → Kunden i situationen →
Utvidga problemet → Välj problemram → Verkligheten → Lösningskärnan →
Det unika och affären → Lindos råd → Slutkort och test), bygger en
personprofil i steg 2 som följer med genom resten av sessionen, och
märker varje påstående i slutkortet med sitt ursprung (användarens egna
ord, Lindos tolkning, eller extern källa med länk). Skärp är en
arbetsyta med ett steg i fokus i stället för en chattlogg: en huvudfråga
per steg och högst en följdfråga, det användaren sagt växer fram i en
idékarta med nio fält i railen (klickbara för att rätta), och Lindo
lyfter insiktsögonblick — en konkret koppling, motsägelse eller ett
mönster i användarens egna ord (efter steg 6 även i det som hittats ute
i världen) — efter steg 3, efter steg 6 och i slutkortet. I steg 6
analyserar Lindo efter användarens reaktion hur idén kan vinna mot de
verkliga konkurrenterna i researchen (svagheter, missat segment, 2–3
sätt att sticka ut).

**Lindos råd (steg 9)** ersätter Skeptikern och "Saker värda att testa":
ett eget, isolerat AI-anrop (`councilAssessment`) med fem röster,
1–2 meningar var och gärna oense — Motsägaren (varför det kan gå fel),
Förstaprincip-tänkaren (kärnan utan antaganden), Optimisten (den dolda
uppsidan), Den utomstående (hur en typisk kund eller en konkurrent ur
researchen reagerar) och Exekutören (det realistiska, vad som görs
först) — och sist Ordföranden med beslut, största risk och första steg.
Märkt "Lindos förslag", inga fakta utanför användarens svar och
researchen, stoppregeln efter rådet som förut. Rådet står i slutkortet
före testkortet, som utvecklar Ordförandens slutsats.

Slutkortet har inga sifferbetyg: det börjar med det som är starkt, ger
varje område "Starkt", "På god väg" eller "Behöver mer underlag" med en
mening om nästa steg, och samlar "Perspektiv du fått av Lindo", Lindos
råd, ett första test och frågor att ta med till rådgivaren. Sedan
Skärpan (§0.13) tänker Lindo med en inbyggd kunskapsbas — som sedan
§0.14 i första hand formar bidragen, inte bara frågorna — tankeverktyg
och kända fallgropar per idétyp, bara heuristiker och aldrig statistik —
och väljer frågor efter informationsvärde för grundare som redan kan
grunderna. Ett tyst klassificeringsanrop efter steg 1 (sessionens enda
extra anrop: 18 logiska anrop utan följdfrågor, tak ~25) bedömer idétyp,
vad som redan är sagt (sagt är inte belagt), vad som är mest osäkert och
avgörande och hur van användaren är, och vanlig kod väljer sedan
deterministiskt vilka verktyg varje steg får i SKÄRPA-blocket. Varje
fråga är EN mening på högst 15 ord, en sak, enkel svenska, med ankaret i
användarens egna ord och visar sitt perspektiv som en diskret etikett,
nya mjuka kontroller stoppar statistik, påhittade namn och siffror,
platta grundfrågor, facktermer och avskrivna exempel, och
kvalitetsevalen i `docs/skarp-kvalitet.md` mäter om det blir skarpare.

**Gränssnittet (§0.15):** ingen fråga, spegling, insikt, förslag eller
rådsröst klipps med "…" (ingen line-clamp/truncate/ellips/fast höjd,
`trimToSentences` behåller hela meningar). Hela steget får plats utan att
sidan scrollar på 1366×768, 1440×900 och mobil — listor, rådet och
idékartan scrollar internt, frågan och svarsytan står fast nertill.
Kartan har kvar två axlar och romben som flyttas, men ingen zoom,
axelnamnen utanför rutan, konkurrenterna som numrerade romber med namnen
i en lista, tydligt var romben är nu och diskret var den har varit. "←
Tillbaka" och klickbara avklarade steg i stegindikatorn visar tidigare
skärmar med svaren förifyllda; oförändrat framåt ger inga nya AI-anrop,
ett ändrat svar kör om de påverkade stegen (med tidigare svar
förifyllda), research körs bara om enligt den befintliga regeln, och
inget sparas dubbelt eller flyttar kartan två gånger.
Se §0 i `docs/skarp-flode.md`, som gäller före resten av dokumentet.
Fullständig specifikation: `docs/skarp-flode.md`.

Kompass-läget är oförändrat och beskrivs inte här.
