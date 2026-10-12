# Skärp-läget — flödesspecifikation (ny version)

Ersätter helt det gamla Skärp-flödet (Riktning → Dina förutsättningar →
Problem → Målgrupp → Konkurrenter → Unikt → Affärsmodell → fast
avslutningsmall). Det gamla finns kvar i git-historiken. Denna fil är
källan CLAUDE.md pekar till för Skärp-läget; `product.md`, `decisions.md`,
`roadmap.md` och `design-reference/HANDOFF.md` är uppdaterade så att de
pekar hit i stället för att beskriva det gamla flödet.

Kompass-läget, designen, API-/AI-lagret (OpenRouter, reservkedjan, tålig
JSON-tolkning, timeouts, Perplexity-providern, research-pipelinen, cachen,
loggningen) och affärsmodellen berörs INTE av detta dokument.

Status: fas 1 (specifikation + plan), väntar på godkännande innan kod
skrivs. Byggs i repot `gxy652fxr` (skarp.functions.ts, ai/schemas.ts,
routes/skarp.tsx m.fl.) — se "Implementationsplan" nedan.

## 1. Principerna, kort

Boggel i Skärp-läget ska vara en tankemotor, inte en idémaskin: gräva fram
användarens eget råmaterial (erfarenheter, irritationsmoment, kontakter,
tillgångar) och strukturera det, inte skriva idéer eller säljtext åt
användaren.

1. **Tankemotor, inte idémaskin.** AI:n frågar, speglar, strukturerar,
   utmanar — skriver aldrig färdiga idéer eller säljtext.
2. **Människan först där det räknas.** I "Du först"-steg skriver
   användaren sitt svar innan AI-förslag visas. Förslag visas efter att
   användaren skrivit, eller om användaren väljer "Jag vet inte — visa
   förslag" (flyktväg). I "AI först"-steg (sökning, strukturering,
   kontroll) går AI:n först.
3. **Personligt hela vägen.** En personprofil byggs i steg 2 och skickas
   med i varje AI-anrop från steg 3 och framåt.
4. **Problem före lösning.** Problemet utforskas (steg 3–5) innan
   lösningen diskuteras (steg 7).
5. **Unikt = en ovanlig, värdefull KOMBINATION** (kund, situation, insikt,
   mekanism, affärsmodell, distribution, grundarens tillgångar) — inte
   "ingen har gjort något liknande".
6. **Evidensmärkning.** Varje påstående har ett ursprung: `anvandare`
   (användarens egna ord), `ai_tolkning` (AI:ns slutsats) eller
   `extern_kalla` (research, med länk). Se §3 för hur det implementeras
   utan att svälla scheman.
7. **Slutar i ett verklighetstest.** Hitta aldrig på siffror, namn,
   marknadsstorlekar eller källor.

## 2. AI-roller (instruktionsmoduler)

Byggs som separata, återanvändbara textblock i `guidance/shared.ts`
(eller en ny `guidance/skarp-roles.ts`, se implementationsplanen), en
modul per roll, som respektive steg monterar in i sin systemprompt:

- **Spegeln** — återger användarens ord, skiljer fakta från tolkning,
  frågar "har jag förstått rätt?". Används i steg 1.
- **Intervjuaren** — öppna frågor om konkreta händelser, en fråga i
  taget. Används i steg 2, 3, 4.
- **Omformuleraren** — alternativa problemramar. Används i steg 5.
- **Linsen** — öppnar lösningsrymden med namngivna tankeoperationer.
  Används i steg 7.
- **Skeptikern** — svaga antaganden och motbevis, ALDRIG samma anrop som
  skapar/sammanfattar idén. Används i steg 9, eget anrop.
- **Syntetiseraren** — sammanfattar enbart från insamlat material, märker
  allt den själv lägger till som `ai_tolkning`. Används i steg 10.

Alla roller ärver samma grundregler (svenska, korta meningar, en fråga i
taget, be om händelser inte etiketter, spegla innan tolkning, hitta
aldrig på siffror/namn/källor) från en gemensam bas — samma honesty-regel
som idag (`ROLE_AND_HONESTY`), men se §9.1 om varför den delas i två
delar.

## 3. Evidensmärkning — implementation utan att svälla scheman

Att be AI:n tagga VARJE mening med `anvandare`/`ai_tolkning`/`extern_kalla`
hade krävt ett ursprungsfält per textsträng i nästan alla scheman — stor
risk för ogiltig JSON hos en gratismodell (bryter begränsning 5). I
stället bestäms ursprunget av **var i datamodellen** en text hamnar,
inte av en AI-ifylld tagg:

- **`anvandare`**: text som är en direkt kopia av vad användaren skrev
  (t.ex. den valda problemramens ordalydelse om användaren behöll sin
  egen, eller ett citat som sparas ordagrant). Appen vet att det är
  användarens eget eftersom det ÄR det sparade svaret, inget AI-anrop
  inblandat.
- **`extern_kalla`**: text som kommer från `research()`-resultat. Länken
  följer alltid med från `ResearchFinding.source` — aldrig AI-genererad.
  Konkurrenter/alternativ som AI:n nämner i steg 6 och 8 matchas tillbaka
  mot de faktiska fynden med samma ordmatchning som redan finns
  (`nameMatchesFinding`/`normalizeWords` i `research/match.ts`, byggd för
  `mapCompetitors`) — hittar AI:n inget matchande fynd behandlas namnet
  som påhittat och filtreras bort, exakt samma skyddsmekanism som
  konkurrentkartan redan har.
- **`ai_tolkning`**: allt annat AI:n skriver själv (Spegelns tolkning,
  Syntetiserarens sammanfattningar, Skeptikerns punkter, Linsens/
  Omformulerarens alternativ). Default-hinken, kräver ingen särskild
  AI-logik.

Praktisk konsekvens: scheman har fält som `fakta`/`tolkning`/`hopp`
(steg 1) eller `direct`/`indirect` (steg 6, namn som matchas mot fynd
efteråt) snarare än en generisk `{text, source}`-lista. Ursprunget är en
egenskap hos FÄLTET, inte hos varje enskild sträng.

**Öppen fråga (se §8):** ska `extern_kalla`/`ai_tolkning` visas som
synliga etiketter i gränssnittet, eller bara genom HUR texten visas
(länk för extern källa, vanlig text för ai_tolkning, kursiv/citat för
användarens egna ord)? Jag föreslår det senare (ingen ny visuell design),
se §8.1.

## 4. Personprofil

Litet, platt objekt, byggs i steg 2, kan uppdateras om nya uppgifter
dyker upp senare. Skickas med i varje AI-anrop från steg 3 och framåt
(som ett kontextblock, samma mönster som `contextBlock`/`heardBlock`
idag). Sparas i sessionens klienttillstånd (React state, som `answers`/
`axes` redan sparas idag) — INTE i Supabase ännu (specen säger uttryckligen
att det "senare kan sparas... per konto", vilket jag tolkar som
uttryckligt framtida arbete, inte något fas 2 ska bygga nu, se §8).
Typad så att Kompass-läget kan återanvända samma typ senare, men Kompass
rör vi inte nu.

```ts
type Personprofil = {
  erfarenheter: { text: string; exempel: string }[]; // max ~5
  fardigheter: { text: string; bevis: string }[]; // max ~5
  intressen: string[]; // max ~5
  natverk: string[]; // max ~5
  tillgangar: string[]; // max ~5
  begransningar: string[]; // max ~5
  blindFlack: string; // "något användaren sett som andra missförstår"
};
```

## 5. Övergripande AI-arkitektur per steg

Två mönster, beroende på om steget är en enskild fråga eller en liten
intervju med flera delfrågor:

**Enkelt steg** (en fråga/ett svar, eller ett val mellan AI-genererade
alternativ): 1 AI-anrop. Reaktionen (`moveX`/`moveY`/`reaction` för
kartan/romben, se §6) läggs i det anropet.

**Intervju-steg** (steg 2, 3, 4 — flera delfrågor i samma makro-steg):
**exakt 2 AI-anrop per makro-steg**, inte ett anrop per delfråga:

1. **Planera** — EN gång, i början av makro-steget: AI:n tittar på vad
   som redan är känt (idé, personprofil, tidigare sparat material) och
   returnerar en ORDNAD LISTA med 1–4 delfrågor (hoppar självmant över
   sådant som redan är besvarat, se steg 4). Användaren svarar sedan på
   frågorna EN I TAGET, i den ordningen — utan nytt AI-anrop mellan
   delfrågorna (ren klient-logik, samma "en fråga i taget" som idag).
2. **Sammanställ** — EN gång, när alla delfrågor i listan är besvarade:
   AI:n väver ihop svaren till makro-stegets strukturerade artefakt
   (personprofil/situationssats/problemkarta) + `moveX`/`moveY`/
   `reaction`.

Detta valdes i stället för att låta AI:n avgöra "räcker det?" efter VARJE
delfråga (vilket hade krävt lika många anrop som delfrågor och en
tillförlitlig AI-bedömning varje gång — svagare mot gratismodeller). Se
§8 om du hellre vill ha den dynamiska varianten.

Totalt per fullständig session: ~15 AI-anrop (steg 1: 2, steg 2: 2, steg
3: 2, steg 4: 2, steg 5: 1, steg 6: 2, steg 7: 1, steg 8: 1, steg 9: 1,
steg 10: 1) — jämförbart med eller färre än dagens 7-stegsflöde.

## 6. Kartan och romben

Fortsätter fungera EXAKT som i dag (begränsning 4), oförändrad mekanism:

- Axlarna sätts en gång, från den rå idétexten, med samma
  `initSkarpMapLogic`/`MapAxesSchema`-anrop som redan finns — oberoende
  av flödets stegindelning (anropas parallellt med steg 1:s Spegel-anrop,
  precis som `initSkarpMap` körs parallellt med första frågan i dag).
- Varje steg som LÅSER något nytt (sammanställningsanropet i
  intervju-steg, eller det enda anropet i enkla steg) fyller i
  `moveX`/`moveY`/`reaction` i SAMMA anrop, exakt som `SkarpJudgeSchema`
  gör i dag — `applyMove`/`toPercent` i `skarp.map.ts` rörs inte.
- Konkurrentkartans placering (`mapCompetitors`, x/y per konkurrent)
  återanvänds OFÖRÄNDRAD i steg 6 (se steg 6 nedan) — samma schema,
  samma huvud→reserv→uppdelning-logik, samma panel.

## 7. Flödet, steg för steg

Alla scheman nedan är skisser (fältnamn/typer) för fas 2 — exakta
`.min()`/`.max()`-gränser sätts vid implementation men ändras inte i
sak. `Suggestion` = befintlig `{label, description, icon, basedOn}`
(oförändrad, se `ai/schemas.ts`).

### Steg 1 — Din idé (Du först, Spegeln)

**Syfte:** fånga idén i användarens egna ord, sortera den i tre listor.
**Vem går först:** användaren (fri textruta, ingen AI innan).
**Roll:** Spegeln.
**AI-anrop:** 2 — (a) `initSkarpMapLogic` (oförändrad, väljer kartans
axlar) parallellt med (b) Spegeln.

```ts
MirrorSchema = {
  mirror: string,        // 1–2 meningar, användarens egna ord
  fakta: string[],       // sådant användaren VET
  tolkning: string[],    // sådant användaren TROR
  hopp: string[],        // sådant användaren ÖNSKAR
}
```

Användaren bekräftar eller rättar (fri text, återanvänder samma
textfält som alltid finns längst ned). En rättelse kör Spegeln EN gång
till på (ursprungstext + rättelse) — fortfarande inom 2 anrop för hela
steget om ingen rättelse behövs, +1 om en rättelse görs.

**Sparas:** idékort v0 = `{ideaText, mirror, fakta, tolkning, hopp}`.
Ingen flyktväg ("visa förslag") i detta steg — Boggel hittar inte på
idéer, så det finns inget att föreslå när rutan är tom.

### Steg 2 — Varför du (Du först, Intervjuaren)

**Syfte:** bygga personprofilen.
**Vem går först:** användaren, på varje delfråga.
**Roll:** Intervjuaren.
**AI-anrop:** 2 (intervju-mönstret i §5).

```ts
PlanQuestionsSchema = {
  questions: { question: string; hint: string; suggestions: Suggestion[] }[], // 1–3
}
PersonprofilResultSchema = {
  personprofil: Personprofil, // §4
  reaction: string,
  moveX: number, moveY: number, moveReaction: string,
}
```

`suggestions` i varje planerad fråga genereras alltid men visas bara om
användaren klickar "Jag vet inte — visa förslag" (flyktväg, §2/§8.2) —
inga extra AI-anrop för flyktvägen.

**Sparas:** `Personprofil`.

### Steg 3 — Kunden i situationen (Du först, Intervjuaren)

**Syfte:** en situationssats.
**Vem går först:** användaren.
**Roll:** Intervjuaren.
**AI-anrop:** 2.

```ts
PlanQuestionsSchema // samma som steg 2, 1–4 frågor (vem/situation/senast/gör idag)
SituationResultSchema = {
  malgrupp: string, situation: string, framsteg: string,
  hinder: string, konsekvens: string, alternativ: string, skal: string,
  reaction: string, moveX: number, moveY: number, moveReaction: string,
}
```

Meningen byggs DETERMINISTISKT i kod (samma mönster som
`renderClosingMarkdown`), inte av AI:n frihand:
`"När {malgrupp} befinner sig i {situation} försöker de {framsteg}, men
{hinder} leder till {konsekvens}. De använder nu {alternativ}, eftersom
{skal}."` — mer robust mot en gratismodell än att be den skriva hela
meningen rätt på en gång.

**Sparas:** situationssatsen (sju fält + den renderade meningen).

### Steg 4 — Utvidga problemet (Du först, Intervjuaren)

**Syfte:** problemkarta (orsaker, aktörer, konsekvenser, större mål).
**Vem går först:** användaren.
**Roll:** Intervjuaren.
**AI-anrop:** 2.

```ts
PlanQuestionsSchema // 1–4 frågor av de 5 kandidat-vinklarna (varför/orsak/vem mer/före-efter/större mål) — AI:n hoppar själv över det som redan framgår av situationssatsen/tidigare svar
ProblemMapResultSchema = {
  orsaker: string[], aktorer: string[], konsekvenser: string[], storreMal: string,
  reaction: string, moveX: number, moveY: number, moveReaction: string,
}
```

**Sparas:** problemkarta.

### Steg 5 — Välj problemram (Du först, sedan Omformuleraren)

**Syfte:** en lösningsneutral problemram.
**Vem går först:** användaren (skriver sin egen ram, fri text — kan
hoppa över via "Jag vet inte — visa förslag direkt", se §8.2).
**Roll:** Omformuleraren.
**AI-anrop:** 1.

```ts
ReframeSchema = {
  reframes: {
    frame: string,
    frameType: "orsak" | "aktor" | "system" | "abstraktion" | "motsats",
    assumption: string,
  }[], // 2–3
  moveX: number, moveY: number, moveReaction: string, // kopplat till ATT välja/behålla en ram, se nedan
}
```

Användaren väljer, kombinerar eller behåller sin egen ram (klientval,
inget nytt AI-anrop). **`moveX`/`moveY`/`reaction` sätts när anropet
görs, baserat på rimlig bedömning av var ramarna generellt pekar** —
en förenkling eftersom det faktiska VALET sker efteråt utan AI-inblandning;
alternativet (ett extra litet anrop EFTER valet, bara för flytten) är
möjligt men bryter "inget nytt anrop för ett klientval" — se §8.3 för om
du hellre vill ha den varianten.

Startar konkurrent-research i bakgrunden här (se §8.4 om VILKET ämne,
`"konkurrenter"` eller `"problem"` — jag föreslår `"konkurrenter"`
eftersom steg 6 konsumerar den), med den valda ramen som sökfråga —
**exakt samma `research()`-funktion, oförändrad** (fire-and-forget,
samma mönster som `prefetchUpcomingResearch` i dagens kod).

**Sparas:** vald problemram (ramtext, lösningsneutral).

### Steg 6 — Verkligheten (AI först, research)

**Syfte:** evidenskarta + konkurrentkartan (oförändrad).
**Vem går först:** AI (research redan igång sedan steg 5).
**Roll:** ingen ny — ren strukturering av research-resultat.
**AI-anrop:** 2 — (a) ny evidenskarta-kategorisering, (b) `mapCompetitorsLogic`
**oförändrad** (samma schema, samma split-på-fel, samma panel/karta).

```ts
EvidenceMapSchema = {
  direct: { name: string }[],   // max 6 — matchas mot research-fynd efteråt (nameMatchesFinding)
  indirect: { name: string }[], // max 6 — samma matchning
  doNothing: string,            // "att inte göra något"-alternativet
  gaps: string[],               // max 4, kunskapsluckor
}
```

Länkarna (`extern_kalla`) hämtas EFTER anropet genom att matcha `name`
mot de faktiska `ResearchFinding`-objekten (samma `nameMatchesFinding`
som konkurrentkartan redan använder för att avvisa påhittade namn) —
AI:n skriver aldrig en URL själv.

Frågan till användaren efter evidenskartan är FAST text, inget AI-anrop:
*"Vad överraskar dig här, och ändrar det något?"* Svaret sparas som fri
text och följer med som kontext framåt.

**Sparas:** evidenskarta (`direct`/`indirect` med länkar, `doNothing`,
`gaps`) + konkurrentkartans resultat (oförändrat) + användarens reaktion.

### Steg 7 — Lösningskärnan (Du först, sedan Linsen)

**Syfte:** mekanismen.
**Vem går först:** användaren ("Om du tar bort alla funktioner, vad är
kärnan...").
**Roll:** Linsen.
**AI-anrop:** 1.

```ts
MechanismSchema = {
  mechanisms: {
    mechanism: string,
    operation:
      | "ta_bort_steg" | "byt_betalare" | "gor_manuellt_forst"
      | "forebygg_i_stallet_for_laga" | "hamta_fran_annan_bransch" | "extremfall",
    rationale: string,
  }[], // 2–3
  moveX: number, moveY: number, moveReaction: string,
}
```

Tankeoperationen (`operation`) visas som en kort prefix-etikett i
förslagets `label` (t.ex. "Hämtat från annan bransch — ...") i den
BEFINTLIGA `SuggestionList`-komponenten, ingen ny komponent (se §8.5).

**Sparas:** mekanismen (användarens egen, eller den valda/justerade).

### Steg 8 — Det unika och affären (Du först, AI kontrollerar)

**Syfte:** särskiljbarhet + affärsgrund, kontrollerat mot evidenskartan.
**Vem går först:** användaren (beskriver kombinationen + svarar på
vem-använder/vem-köper/vem-betalar/första-kanal, en fri text).
**Roll:** ingen namngiven roll i specen — ren kontroll-logik mot
evidenskartan, i kärn-AI:ts röst.
**AI-anrop:** 1.

```ts
UniqueAndBusinessSchema = {
  uniqueCombination: string,       // AI:ns strukturerade återgivning (ai_tolkning)
  conflictWarning: string | null,  // flaggar om något "unikt" redan görs av en konkurrent i evidenskartan
  anvandare: string, kopare: string, betalare: string, forstaKanal: string,
  moveX: number, moveY: number, moveReaction: string,
}
```

Evidenskartan skickas med i kontexten; AI:n gör jämförelsen själv
(semantisk, inte en ren namnmatchning som i steg 6 — jämförelsen "är
detta redan unikt" är för resonemangsbaserad för en mekanisk matchning).

**Sparas:** särskiljbarhet + affärsgrund.

### Steg 9 — Skeptikern (AI först, eget anrop)

**Syfte:** antagandekarta + stoppregel.
**Vem går först:** AI (Skeptikern).
**Roll:** Skeptikern — **MÅSTE vara ett eget anrop**, aldrig samma som
skapar eller sammanfattar idén (uttrycklig speckrav).
**AI-anrop:** 1, helt isolerat (tar emot allt sparat material, men
skriver INGET annat än kritik — inga förslag, ingen sammanfattning).

```ts
SkepticSchema = {
  concerns: {
    type:
      | "svagaste_antagande" | "motbevis" | "varfor_inte_redan_lost"
      | "kopieringsrisk" | "ai_nodvandig_eller_pynt",
    point: string,
  }[], // 1–3 — AI:n väljer de mest relevanta, inte alla fem varje gång
}
```

Frågan efter är FAST text, inget AI-anrop: *"Vad skulle få dig att byta
riktning eller lägga ner?"* Svaret blir stoppregeln direkt (sparas
ordagrant, `anvandare`).

**Sparas:** antagandekarta (`concerns`) + stoppregel (användarens egna
ord).

### Steg 10 — Slutkort och test (Syntetiseraren)

**Syfte:** ersätter den gamla avslutningsmallen helt.
**Vem går först:** AI (Syntetiseraren), bygger ENBART från sparat
material.
**Roll:** Syntetiseraren.
**AI-anrop:** 1 (se §8.6 om varför detta är det STÖRSTA schemat i det
nya flödet, och reservplanen om det strular).

```ts
ClosingCardSynthesisSchema = {
  ideaOneLiner: string, // ai_tolkning — finns inte ordagrant sparat sen tidigare
  strengthFactors: {
    factor:
      | "problemevidens" | "grundarpassning" | "losningens_skillnad"
      | "genomforbarhet" | "affarsmojlighet" | "larhastighet",
    score: number,          // 1–5
    certainty: "lag" | "medel" | "hog",
    basis: string,          // vad poängen bygger på
  }[],                       // exakt 6, ingen totalsumma
  testCard: {
    hypothesis: string, method: string, signal: string, within48h: string,
  },
  advisorQuestions: string[], // exakt 3
}
```

De fält som REDAN finns ordagrant sparade — problemramen (steg 5),
situationssatsen (steg 3), mekanismen (steg 7), unikt+affärsgrund
(steg 8), svagaste antagandet+stoppregeln (steg 9) — sätts in i den
färdiga kortet DIREKT I KOD, utan nytt AI-anrop (samma mönster som
`renderClosingMarkdown` i dag: AI:n fyller bara i det som FAKTISKT är
nytt, koden bygger ihop helheten deterministiskt). Detta håller
AI-anropet litet trots att slutkortet är rikt.

Slutkortet renderas med den BEFINTLIGA `ResultReport`-komponenten,
oförändrad — bara en ny markdown-struktur (rubriker + punktlistor,
samma parsningsmönster som redan finns, se §8.7).

**Sparas:** hela slutkortet (markdown-text, som `resultText` i
`saveSession`, oförändrad signatur).

Användaren kan rätta slutkortet (fri text → ett nytt, litet
synteskorrigering-anrop, samma mönster som steg 1:s rättelse).

## 8. Oklarheter, risker och frågor till dig

### 8.1 Evidensmärkning — synliga taggar eller bara stil?

Jag föreslår att ALDRIG visa en explicit etikett som "[extern_kalla]" i
gränssnittet (ingen ny visuell design) — i stället särskiljs ursprung
genom hur texten visas: länk för extern källa (redan ett mönster i
appen, se konkurrentpanelens råa fynd), vanlig text för ai_tolkning,
ev. kursiv/citat för direkta användarcitat. Vill du i stället ha
explicita taggar krävs ett litet, nytt UI-element (en kort etikett per
rad) — säg till.

### 8.2 Flyktvägens omfattning

Jag tolkar "en flyktväg så att ingen fastnar" som att den gäller
rena VAL-steg (5: problemram, 7: mekanism) och som ett enkelt
exempel-stöd på intervjufrågor (2, 3, 4, 8) — INTE steg 1 (inget att
föreslå när Boggel inte hittar på idéer). Säg till om du vill ha den
bredare eller smalare.

### 8.3 Kartans rörelse i val-steg (5, 7)

I steg 5 och 7 sker det FAKTISKA valet client-side (ingen AI-inblandning
när användaren klickar ett alternativ) men `moveX`/`moveY` behöver
komma från NÅGOT AI-anrop för att romben ska flytta sig korrekt utifrån
VILKET alternativ som faktiskt väljs. Jag föreslår att anropet som
genererar alternativen (Omformuleraren/Linsen) SKA sätta en rörelse
redan då, generellt grundad i vad alternativen som grupp pekar mot —
INTE en exakt rörelse per specifikt val. Alternativet är ett extra litet
AI-anrop EFTER att användaren valt (för en exakt rörelse per val) — det
håller budgeten på "max 1–2 anrop per stegövergång" men lägger till ett
tredje anrop just i de två stegen. Säg till vilket du föredrar.

### 8.4 Vilket forskningsämne startar steg 5?

Specen säger "Starta problem-research i bakgrunden" i steg 5, men steg 6
("Verkligheten": direkta/indirekta alternativ) behöver
konkurrent/alternativ-data — samma sak som dagens `mapCompetitors`
(`topic: "konkurrenter"`). Jag tolkar specens ordval som "sök utifrån
den valda problemramen", inte bokstavligen `topic: "problem"`, och
föreslår att steg 5 startar `topic: "konkurrenter"`-sökningen (det steg
6 faktiskt konsumerar). Vill du att BÅDA ämnena förhandshämtas i steg 5
(t.ex. för att även mata steg 8:s kontroll), eller att steg 4 redan då
startar `topic: "problem"` (eftersom steg 4 explicit handlar om att
utvidga/bevisa problemet)? Säg till.

### 8.5 Tankeoperation/ramtyp som etikett i förslag

Jag föreslår att visa ramtyp (steg 5) och tankeoperation (steg 7) som en
kort prefix i `Suggestion.label` (t.ex. "Motsats — ...") i den
BEFINTLIGA `SuggestionList`-komponenten, utan att ändra komponenten.
Alternativet är att lägga till ett litet `tag`-fält i `Suggestion` och
rendera det som en liten etikett i listan — en minimal men ÄKTA
komponentändring (en ny rad kod i `SuggestionList.tsx`, ingen ny
design). Säg till vilket du föredrar.

### 8.6 Steg 10:s schema är det största i det nya flödet

`ClosingCardSynthesisSchema` (sex styrkefaktorer + testkort + tre
rådgivarfrågor) är jämförbar i storlek med dagens `ClosingSchema`, men
något större. Om testet i fas 2 visar att gratismodellen har svårt med
det delar jag upp det i två anrop (styrkeprofilen för sig, testkort +
rådgivarfrågor för sig) — vill du att jag förbereder den uppdelningen
direkt i fas 2 i stället för att vänta och se om det behövs?

### 8.7 `ResultReport`-rendering av steg 10

Bekräftat: `ResultReport.tsx` parsar generisk markdown (`**Rubrik**` +
punktlistor) och kräver INGEN kodändring för den nya slutkorts-
strukturen — bara en ny `renderClosingCardMarkdown`-funktion
(motsvarande dagens `renderClosingMarkdown`). Ingen fråga här, bara
dokumenterat för tydlighetens skull.

### 8.8 Den gamla "bygg_på"-kvalitetsgrinden tas bort

Det gamla flödets "bygg_på"-mekanik (upp till 2 försök, be om en bättre
formulering) nämns inte i den nya specen. Jag tar bort den helt —
Intervjuaren ställer i stället upp till N frågor (planerade i förväg,
§5) och ber om konkreta exempel som en AV dessa frågor, inte som en
separat kvalitetsgrind. Säg till om du vill behålla något liknande en
retry-mekanik för uppenbart tomma/mycket korta svar.

### 8.9 "Få en handlingsplan"-knappen

Dagens valfria knapp efter slutresultatet (`synthesizeActionPlan`, en
tidsatt veckoplan) överlappar nu innehållsmässigt med det nya
Testkortet (hypotes/metod/signal/48-timmarshandling) i steg 10. Ska jag:
(a) ta bort knappen helt (Testkortet täcker behovet), (b) behålla den
som ett valfritt, LÄNGRE tillägg (flera veckors uppföljning, bortom de
första 48 timmarna), eller (c) slå ihop dem? Jag lutar åt (b) om du
inte säger annat, eftersom Testkortet är en 48-timmarsåtgärd och den
gamla knappen är en flerveckorsplan — olika tidshorisont, inte
dubblett.

### 8.10 Personprofil-persistens

Specen säger att personprofilen "sparas i sessionens tillstånd (så att
den SENARE kan sparas i Supabase per konto)". Jag tolkar det som att
fas 2 INTE ska bygga kontobunden persistens nu — profilen lever i
klientens React-state under sessionen, precis som `answers`/`axes` gör
idag, och (för kontinuitet) skickas med i den slutliga `saveSession`
som ytterligare en `{step, answer}`-post, utan schemaändring i
Supabase. Säg till om du vill att jag förbereder en databaskolumn redan
nu.

### 8.11 Pre-existerande dokument-inkonsekvenser (inte en del av denna
uppgift, flaggas ändå)

- `CLAUDE.md` pekar på `docs/product.md` m.fl., men filerna ligger i
  repots ROT (`product.md`, inte `docs/product.md`) — en äldre
  path-förskjutning, inte orsakad av detta arbete. Jag har lagt den nya
  filen på den path CLAUDE.md FAKTISKT anger (`docs/skarp-flode.md`) för
  konsekvens, men rört inte de andra filernas placering. Säg till om du
  vill att jag flyttar `product.md`/`architecture.md`/`roadmap.md`/
  `decisions.md` in i `docs/` också, så CLAUDE.md:s paths stämmer
  överallt.
- `architecture.md` beskriver kärn-AI:t som "Claude API", men den
  faktiska implementationen (sedan tidigare arbete) kör OpenRouter/
  Gemma via ett OpenAI-kompatibelt lager. Detta rör varken Skärp-flödet
  eller AI-lagrets funktion, så jag har INTE ändrat det — men flaggar
  det eftersom det är felaktigt dokumenterat.

## 9. Implementationsplan (fas 2)

### 9.1 Filer som ändras, i ordning

1. **`src/lib/ai/schemas.ts`** (repo `gxy652fxr`) — lägg till nya
   scheman (`MirrorSchema`, `PersonprofilSchema`, `PlanQuestionsSchema`,
   `SituationResultSchema`, `ProblemMapResultSchema`, `ReframeSchema`,
   `EvidenceMapSchema`, `MechanismSchema`, `UniqueAndBusinessSchema`,
   `SkepticSchema`, `ClosingCardSynthesisSchema`) + en
   `renderClosingCardMarkdown`-funktion. `SkarpJudgeSchema`/
   `SkarpJudgeWithNextSchema` tas bort (bara använda av Skärp, ersätts
   helt). Allt Kompass använder (`JudgeSchema`, `OpeningSchema`,
   `StepQuestionSchema`, `ClosingSchema`, `ActionPlanSchema`,
   `CompetitorMapSchema`/`CompetitorPlacementSchema`,
   `SuggestionSchema`) lämnas ORÖRT.
2. **`src/lib/guidance/shared.ts`** — dela `ROLE_AND_HONESTY`: flytta
   meningen "Du håller dig strikt inom det styrda flödets struktur
   (menyval med fritext som override)" ut ur den delade konstanten och
   in i Kompass egen `BASE`-prompt ORDAGRANT (Kompass egen slutprompt
   blir identisk med i dag — ingen beteendeändring för Kompass). Lägg
   till de sex rollmodulerna (Spegeln/Intervjuaren/Omformuleraren/
   Linsen/Skeptikern/Syntetiseraren) som nya exporterade konstanter,
   antingen här eller i en ny `guidance/skarp-roles.ts`.
3. **`src/lib/skarp.functions.ts`** (eller en ny `src/lib/skarp/`-mapp,
   se nedan) — rewrite: nya `STEPS` (10 steg, nya id:n), nytt `Personprofil`-
   state, nya `*Logic`-funktioner per steg (enligt §7), nya
   `createServerFn`-wrappers som bara delegerar till dem (samma mönster
   som idag efter förra omstruktureringen). `mapCompetitorsLogic`,
   `research()`-anropen, och `generateValidated`/`generateStructured`
   används OFÖRÄNDRADE.
   - **Föreslagen filuppdelning** (filen växer annars för mycket): dela
     upp i `src/lib/skarp/types.ts` (STEPS, Personprofil, delade typer),
     `src/lib/skarp/steps/*.ts` (en fil per steg eller gruppering:
     mirror, interview (2–4 delar samma intervju-mönster),
     reframe+evidence+mechanism, unique+skeptic+closing),
     `src/lib/skarp/server-functions.ts` (alla `createServerFn`-
     wrappers, importerade av `routes/skarp.tsx` precis som idag). Säg
     till om du hellre vill hålla allt i en enda fil som i dag.
4. **`src/components/shared/StepRow.tsx`** — uppdatera `SHORT_LABELS`
   med de tio nya steg-etiketternas förkortningar (additivt, påverkar
   inte Kompass etiketter eftersom de har andra nycklar).
5. **`src/routes/skarp.tsx`** — den stora UI-omskrivningen: nytt
   `Entry`-typ-schema för de nya innehållstyperna (mirror-lista,
   problemkarta, evidenskarta, slutkort), ny klient-logik för
   intervju-mönstrets "planera N frågor, stega igenom dem", ny
   "Jag vet inte — visa förslag"-länk (återanvänder befintlig
   underline-länkstil), personprofil/situationssats/problemkarta/
   evidenskarta i React state. Kartan/romben (`SkarpMap`/`SkarpRail`)
   återanvänds OFÖRÄNDRADE.
6. **`scripts/skarp-flow-test.ts`** — uppdatera till de nya *Logic-
   funktionerna, simulera "Du först" (skriv ett rimligt svar INNAN
   eventuella förslag hämtas/visas) och intervju-mönstrets flerstegs-
   frågor.
7. Verifiera: `npx tsc`, `npx eslint .`, `npm run build`,
   `npx tsx scripts/skarp-flow-test.ts "..."` mot mock-AI, och en
   snabb dev-server-genomgång (samma Playwright-rök-test-mönster som
   använts tidigare den här sessionen) innan push.

### 9.2 Gränssnittsbeteenden som ändras (begränsning 1)

1. **Förslag göms tills användaren skrivit, eller tills "Jag vet inte —
   visa förslag" klickas.** Löses med en ny boolesk per-fråga-flagga i
   klientens state + en liten länk i BEFINTLIG understruken text-länk-
   stil (samma klasser som redan används för "Försök igen"/"Börja om").
   Ingen ny komponent.
2. **Flerfråge-steg (2, 3, 4) stegar igenom 1–4 delfrågor innan
   StepRow-diamanten för det steget låses.** Samma fråga/svar-rendering
   som i dag (`kind: "question"`/`"answer"` entries), bara fler i rad
   innan steget räknas som klart. Ingen ny komponent.
3. **Steg 1 visar en spegling + tre listor (Fakta/Tolkning/Hopp).**
   Renderas som vanlig markdown i den BEFINTLIGA "assistant"-stilen
   (`ReactMarkdown`, samma klasser som handlingsplanen redan använder).
   Ingen ny komponent.
4. **Steg 5/7 visar AI-alternativ med en ramtyp/tankeoperation-etikett.**
   Löses som en prefix i `label`-texten i den BEFINTLIGA
   `SuggestionList` (se §8.5 för alternativet med ett nytt `tag`-fält).
5. **Steg 6 visar en evidenskarta (direkta/indirekta alternativ, "gör
   inget", kunskapsluckor) UTÖVER den befintliga konkurrentkartan.**
   Renderas som vanlig markdown med länkar för externa källor — samma
   länkstil som konkurrentpanelens råa fynd redan har. Konkurrentkartan/
   panelen (`SkarpMap`/`SkarpRail`) rörs inte.
6. **Steg 10:s slutkort har en helt ny innehållsstruktur** men
   återanvänder `ResultReport` OFÖRÄNDRAD (se §8.7).
7. **StepRow visar 10 steg + "Slutkort" i stället för 7 + "Komplett
   beskrivning".** Samma komponent, bara fler/andra etiketter — raden
   har redan horisontell scroll för trånga skärmar.

Ingen av dessa kräver en NY visuell komponent eller nytt färg-/typsnitt-
beslut — alla återanvänder befintliga mönster, bara med ny data/innehåll
och enkla state-/villkorsändringar.

## 10. Sammanfattning för CLAUDE.md

En kort version av §1–§2 och en pekare hit läggs i `CLAUDE.md` (fas 1,
redan gjort i samma commit som denna fil).
