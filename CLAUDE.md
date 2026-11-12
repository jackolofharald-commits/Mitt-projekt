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
jag-form, egna åsikter som sägs rakt, minns vad användaren sagt och
refererar till det naturligt, varm, nyfiken, rak och lite lekfull — som
en kompis som kan affärer, över en kopp kaffe. Utan maner: "Ärligt talat
tror jag …" och "Om jag var du …" högst en gång per steg och aldrig i
kvittensen, citattecken runt användarens ord bara i insiktsögonblicken,
och hypoteser låter som hypoteser ("kanske", "jag gissar"). Se §0.15 och
§0.16 i `docs/skarp-flode.md`.

Skärp-läget är INTE ett styrt menyflöde längre (det var det gamla
flödet). Grundprincipen är GE FÖRST, FRÅGA SEN (§0.14): Lindo är en
expert som tänker MED användaren och tillför mer än användaren själv
gör. **Varje steg har exakt tre delar (§0.16):** (1) en **kvittens** —
EN mening om vad Lindo tog med sig från förra svaret, inte en
sammanfattning av idén; (2) **Lindos tankar** — 1–3 kort, var och ett
en konkret rubrik (högst sex ord, aldrig ett verktygsnamn) och EN mening,
märkta "Lindos förslag" och visade med Boggels grafiska profil (gul
accent, romber, papperskänsla, mjuk animation); (3) **EN fråga** med 2–4
klickbara svar, "Vet inte" och fritext. Inga upprepningar av idén —
idékartan visar helheten. Korten tas fram brett och visas smalt: modellen
föreslår cirka sex kandidater och koden sållar bort det som kunde stått
om vilken idé som helst, bara upprepar ett ramverk, redan sagts eller
motsäger idémodellen; varje kort har något konkret (ett exempel, ett
namngivet kundsegment, något ur marknadsbilden, ett första drag), högst
ett riskkort per steg. Frågorna handlar om idén (problem, kund, värde,
skillnad mot alternativen, affär), aldrig om genomförandets logistik.
Användaren skriver själv först BARA i steg 1 (idén) och i steg 5 och 7
(egen problemram och kärna) innan Lindos förslag visas. Den gamla
principen — "tankemotor, inte idémaskin", "AI:n frågar, speglar,
strukturerar och utmanar", "skriver aldrig färdiga idéer", "Du först" i
alla steg — gäller INTE längre och ska inte återinföras.

**Steg 1 loopar inte:** Lindo speglar idén i 1–2 meningar som bara
återger det användaren sagt (inga bedömningar som "underlag saknas") och
frågar en gång "Har jag fattat rätt?" — "Ja, vidare" eller "Lägg till
något". Ett tillägg kvitteras kort överst i steg 2 ("Uppfattat – jag tar
med att …") och flödet går vidare; högst en rättelserunda, senare
tillägg skrivs i nästa steg. Fakta/tolkning/hopp sorteras bara internt.

**Frågeregeln (§0.16):** varje fråga ska gå att svara på på tio sekunder
utan research och utan kunskap om en viss person eller ett visst
företag. Huvudformen är Lindos egen hypotes + reaktion ("Jag tror det
svåra är tajmingen: materialet blir över en viss vecka, men köparna
behöver det en annan. Stämmer det med vad ni sett?" med "Ja, det är
kärnan", "Delvis", "Nej, det är något annat"); en sak, högst 15 ord,
vardaglig svenska, inga dubbelfrågor, aldrig "vilka kan ni ringa i
morgon" eller "vilka kontakter har ni". Steg 2 "Varför du" handlar om
användarens fördel (erfarenhet, insikt, åtkomst), aldrig en ringlista.
"Vet inte" är alltid okej: Lindo ger då sin bästa gissning, märkt som
gissning, och går vidare.

**Idémodellen (§0.16):** Lindo har en löpande bild av idén i
`ctx.ideaModel` — vem som säljer, köper, betalar och bestämmer,
kärnhypoteserna och första kundgruppen — som varje steg uppdaterar dolt och som skickas med i
varje anrop tillsammans med det användaren sagt och det Lindo redan
föreslagit. Lindo får inte motsäga sig själv utan att säga att den
ändrar sig och varför, och samma perspektiv kommer inte tillbaka, inte
heller i en rättelserunda.

**Researchen: två körningar i bakgrunden (§0.17):** den breda när idén
skickas (parallellt med speglingen) och den riktade när steg 4 är klart
(första kundgruppen och situationen, §0.18) — användaren väntar aldrig
på dem. Perplexitys
Agent API får `input`, `instructions`, `tools` (web_search med svensk
sökplats, fetch_url för konkurrenternas egna sajter) och `max_steps`
(`max_tool_calls` finns inte i API:t); sökplanerna kräver flera separata
sökningar (direkta konkurrenter, indirekta alternativ, svensk/nordisk
marknad, hur målgruppen pratar, misslyckade försök, regler och trender).
Produkter före forskning; källans slag sätts i kod av domänen (egen sajt
= vad företaget säger om sig självt, forum/recensioner = ledtrådar,
myndigheter/studier = fakta, forskning = komplement); agentens lista
godtas bara för länkar som finns bland sökträffarna — inga påhittade
företag. Ett AI-anrop per körning kondenserar fynden till en kort, tät
marknadsbild (direkta konkurrenter, indirekta alternativ, hur målgruppen
pratar, misslyckade försök, regler och trender, kunskapsluckor) med
hänvisningar till fynden, och den följer med i alla senare anrop. Lindos
kort i steg 2–8 får bygga på ett fynd och visar då "Hittat på nätet" med
länk. Steg 6 läser bara cachen (ingen ny sökning) och visar fynden
grupperade — direkta, indirekta, om kunden inte gör något — med en
mening och en länk var. Den breda får ett omtag om idén ändras i grunden
i steg 1, den riktade om valet av första kundgrupp i steg 4 ändras. Servern
har ett eget tak för researchen (per instans) och kör bara frågor i
Skärps format; webbtext och användarens svar är material, aldrig
instruktioner, och en direkt konkurrent måste stå i fyndets rubrik eller
domän (§0.17 punkt 9). Kompass anrop är oförändrade.

**Första kundgruppen (§0.18):** Lindo hjälper användaren att smalna av
till den kundgrupp där problemet är mest akut och som kan och vill
betala — utan nya steg och utan nya anrop. Steg 3 är oförändrat
(situationen); ett av korten där får vara "Ett spår att ha i
bakhuvudet: …" när Lindo ser en uppenbart intressantare grupp. I steg 4
resonerar korten om var problemet är mest akut (hur ofta, vad det
kostar, om de redan betalar, om de får bestämma, hur lätta de är att
nå) och frågan byggs i kod: "Jag tror problemet är störst för …,
eftersom …. Stämmer det?" med 2–3 smalare grupper, "Behåll min
kundgrupp", "Vet inte" och fritext — en avsmalning, inget byte om inte
användaren väljer det. Valet tolkas i kod när det är ett klick, sparas
i `ctx.ideaModel.forstaKundgrupp` (grupp, varför, källa), syns i
idékartans kundfält ("Börjar med: …"), problemramen i steg 5 och testet
formuleras för den, och den riktade researchen söker på den. Steg 8 får
1–2 kort (från steg 7:s anrop) som stämmer av gruppen mot researchen och
föreslår en första väg att nå den, och frågan "Är det här gruppen du
börjar med?" — "Ja, jag börjar där" eller "Hellre …". Slutkortet har
raden "Första kundgrupp" med en mening om varför. Svaren skickas med
Lindos hypotes och de visade förslagen.

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
mening om nästa steg, visar första kundgruppen med en mening om varför,
och samlar "Perspektiv du fått av Lindo", Lindos
råd, ett första test och frågor att ta med till rådgivaren. Sedan
Skärpan (§0.13) tänker Lindo med en inbyggd kunskapsbas — som sedan
§0.14 i första hand formar bidragen, inte bara frågorna — tankeverktyg
och kända fallgropar per idétyp, bara heuristiker och aldrig statistik —
och väljer frågor efter informationsvärde för grundare som redan kan
grunderna. Ett tyst klassificeringsanrop efter steg 1 bedömer idétyp,
vad som redan är sagt (sagt är inte belagt), vad som är mest osäkert och
avgörande och hur van användaren är, och vanlig kod väljer sedan
deterministiskt vilka verktyg varje steg får i SKÄRPA-blocket.
Anropsbudgeten är 20 logiska anrop utan följdfrågor (18 + två
marknadsbilder), 21 med ett tillägg i steg 1, tak ~25. Frågan visar sitt
perspektiv som en diskret etikett, mjuka kontroller stoppar statistik,
påhittade namn och siffror, platta grundfrågor, facktermer och avskrivna
exempel, och kvalitetsevalen i `docs/skarp-kvalitet.md` mäter om det
blir skarpare.

**Gränssnittet (§0.15, §0.16):** ingen fråga, spegling, insikt, förslag
eller rådsröst klipps med "…" (ingen line-clamp/truncate/ellips/fast
höjd, `trimToSentences` behåller hela meningar). Steget är EN kolumn som
scrollar naturligt — inga inre scrollrutor — med frågan och svarsytan
lätta att se; kartan i högerkolumnen är alltid helt synlig och står
still i alla steg (railen kan inte scrollas, idékartan scrollar bara sin
egen lista). Första skärmen visas direkt: inga AI-anrop innan användaren
skrivit något, och tunga delar (3D-nålen på startsidan) laddas först när
de behövs. Kartan har kvar två axlar och romben som flyttas, men ingen zoom,
axelnamnen utanför rutan, konkurrenterna som numrerade romber med namnen
i en lista, tydligt var romben är nu och diskret var den har varit. "←
Tillbaka" och klickbara avklarade steg i stegindikatorn visar tidigare
skärmar med svaren förifyllda; oförändrat framåt ger inga nya AI-anrop,
ett ändrat svar kör om de påverkade stegen (med tidigare svar
förifyllda), research körs bara om enligt regeln ovan, och inget sparas
dubbelt eller flyttar kartan två gånger.
Se §0 i `docs/skarp-flode.md`, som gäller före resten av dokumentet.
Fullständig specifikation: `docs/skarp-flode.md`.

Kompass-läget är oförändrat och beskrivs inte här.
