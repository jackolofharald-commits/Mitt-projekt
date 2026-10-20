# Skärp-läget — flödesspecifikation (ny version)

Ersätter helt det gamla Skärp-flödet (Riktning → Dina förutsättningar →
Problem → Målgrupp → Konkurrenter → Unikt → Affärsmodell → fast
avslutningsmall). Det gamla finns kvar i git-historiken. Denna fil är
källan CLAUDE.md pekar till för Skärp-läget; `docs/product.md`,
`docs/decisions.md`, `docs/roadmap.md` och `design-reference/HANDOFF.md`
är uppdaterade så att de pekar hit i stället för att beskriva det gamla
flödet.

Kompass-läget, designen, API-/AI-lagret (OpenRouter, reservkedjan, tålig
JSON-tolkning, timeouts, Perplexity-providern, research-pipelinen, cachen,
loggningen) och affärsmodellen berörs INTE av detta dokument.

Status: fas 3 — alla tio steg är ombyggda, i repot `gxy652fxr`
(`src/lib/skarp/`, `src/components/skarp/`, `src/routes/skarp.tsx`).
**§0 gäller.** §1–§10 beskriver fas 1–2 och står kvar som historik; där
de krockar med §0 gäller §0.

## 0. Fas 3 — upplevelse och konstruktion (alla tio steg ombyggda)

Fas 2 fungerade tekniskt men upplevdes som en promptad AI-chatt och ett
förhör. Fas 3 bygger om upplevelse, konstruktion, ton och design i alla
tio steg; stegens syfte och ordning (§7) ligger kvar. Steg 1–3 byggdes
först som provexemplar, med välkomnandet, idékartan och det första
insiktsögonblicket. Steg 4–10 är, efter användarens godkännande,
ombyggda på samma sätt, med insikter även efter steg 6 och i slutkortet.

**Den här sektionen ersätter §5 och §7 helt** — planera/sammanställ-
mönstret finns inte längre, och §7 står kvar som historik. Den avgör
också §8.1, §8.3, §8.5, §8.6 och §8.7 och ändrar §1 punkt 2/§8.2 (se
0.4, 0.8 och 0.11). §1–§4 gäller i sak där §0 inte säger annat. Kartan
och romben (§6), Kompass-läget och AI-/research-lagret är orörda.

### 0.1 Arbetsyta, inte chattlogg

- Ett steg i taget i fokus: kicker "Steg n av 10 · <namn>" (t.ex. "STEG
  1 AV 10 · DET HÄR HÖRDE JAG"), ev. en lead (speglingen av förra
  svaret, 0.5), frågan som rubrik, en rad om varför vi frågar, sedan
  svarsytan (`StepScreen` + `AnswerComposer`). Medan Boggel arbetar står
  "· Boggel läser …" i kickern, på en följdfråga "· om du vill" (0.3).
  Ingen scrollande logg av frågor och svar, inga chattbubblor — inte
  heller efter slutkortet (0.11).
- Det användaren redan sagt syns som innehåll i idékartan (0.2), inte
  som historik. Railen visar under kartan en notisrad (senaste
  speglingen + kartrörelsen i ord via `mapMoveNote`, "flyttar dig mot …
  och mot …") och idékartan, i stället för `HeardLog`. I slutkortet
  zoomas kartan ut.
- Steg x av 10 och ungefär hur lång tid som är kvar står i headern
  (`SkarpProgress`: tio segment, "ca N min kvar", "Nästa: …"). Tiden
  kommer från `STEP_MINUTES` i `progress.ts` (2, 2, 1,5, 1,5, 1,5, 1,5,
  1,5, 1, 1, 1,5 — totalt 15 minuter; slutkortet tar längst att vänta
  in och läsa).
- Vid väntan: `ThinkingLine` — små romber som fylls i tur och ordning och
  korta texter om vad som händer (byts var 2,6 s och stannar på den
  sista; efter 12 s läggs "Det tar lite längre än vanligt — jag är kvar."
  till). Den första texten citerar början av användarens svar (`Läser
  "…"`) där det finns ett; texterna för steg 4–10 står i 0.9. Aldrig en
  tom skärm. Fel visas lugnt i `FocusError` ("Svaret kom inte fram.")
  med "Försök igen"; det användaren skrivit ligger kvar.

### 0.2 Idékartan

- Nio fält (`IDEA_MAP_FIELDS` i `types.ts`). Ligger i railen på desktop
  och som hopfällbar rad på mobil — helt skild från kartan och romben.
- Varje fält fylls efter sitt steg med en kort formulering som bygger på
  användarens egna ord, med det ordagranna citatet guldmarkerat och en
  diskret ursprungsetikett (0.8):

| Fält | Steg | Fylls med |
| --- | --- | --- |
| Idén | 1 | `ideaShort` |
| Du | 2 | `duShort` |
| Kunden och situationen | 3 | `kundShort` |
| Problemet | 4 | `problemShort` |
| Problemramen | 5 | den valda ramen (egen/omskriven = "Dina ord") |
| Verkligheten | 6 | `realityShort` |
| Kärnan | 7 | den valda kärnan (egen/omskriven = "Dina ord") |
| Det unika och affären | 8 | `uniqueShort` |
| Att testa | 9 | första saken värd att testa + "Stoppregel: …" |

- Kartrörelsen står i notisraden under kartan, inte i fälten.
  Ifyllnaden ska kännas som en belöning: fältet blinkar guld, romben
  poppar och "{n} av 9" räknas upp. Vid en följdfråga fylls fältet
  preliminärt (streckad romb) och slutgiltigt efter svaret. "Att testa"
  fylls direkt när stoppregeln är skriven (inte tidigare), med
  stoppregeln som citat — bara när användaren skrivit den själv. Ett
  klickat förslag blir inget citat och märks "Boggels tolkning" i
  slutkortet.
- Användaren kan klicka på ett fyllt fält och rätta det. Rättelserna
  sparas i `ctx.ideaMap` (BARA användarens rättelser — sparas texten
  oförändrad är det ingen rättelse, och fältet behåller sin märkning) och går med i
  varje anrop som ett `IDÉKARTAN`-block som gäller före äldre detaljer.
- Insikterna (0.6) listas under "INSIKTER"; en ny insikt lyser upp och
  scrollas fram som ett nyss ifyllt fält, en avvisad visas överstruken.

### 0.3 En huvudfråga per steg, högst en följdfråga

- Varje steg har EN huvudfråga: en mening (högst 12 ord, ett "?") plus
  en kort rad om varför vi frågar (högst 14 ord). Frågan väljs efter vad
  som SAKNAS i idén och tidigare svar, aldrig ur en fast lista, och
  genereras i föregående stegs anrop:

| Steg | Frågan kommer från |
| --- | --- |
| 2 | `readIdea` |
| 3 | `answerWhyYou` |
| 4 | `answerCustomer` (`nextQuestion`) |
| 5 | `answerProblem` (`frameQuestion`, öppen fråga) |
| 6 | `buildReality` (`question`, byggd på det som hittades) |
| 7 | `answerReality` (`mechanismQuestion`, öppen fråga) |
| 8 | `proposeMechanisms` (`nextQuestion`) |
| 9 | `skepticAssessment` (`stopQuestion`, stoppregeln) |

- I steg 6 och 9 står frågan under det AI:n visat, som en andra fråga på
  samma skärm (`QuestionBlock`, kicker "DIN TUR").
- Högst en följdfråga, och bara när svaret inte säger NÅGOT om stegets
  kärna (steg 2: varför just du; steg 3: vem och i vilken situation;
  steg 4: problemet; steg 6: verkligheten; steg 8: det huvudfrågan
  frågade om). Ett kort svar, en etikett eller ett valt förslag är
  alltid fullgott. Följdfrågan visas med "· om du vill" och har alltid
  "Gå vidare utan →", som använder det preliminära resultatet. När den
  är besvarad tvingar servern `needsFollowUp = false`. Valen i steg 5/7
  och stoppregeln har ingen följdfråga.
- Inget steg planerar längre flera frågor (§5 är ersatt). Anrop per steg:
  steg 1 = `initSkarpMap` + `readIdea` (+1 per rättelse), steg 2 =
  `answerWhyYou`, steg 3 = `answerCustomer` + `findInsight`, steg 4 =
  `answerProblem`, steg 5 = `reframeProblem`, steg 6 = `buildReality` +
  `mapCompetitors`, sedan `answerReality` + `findInsight`, steg 7 =
  `proposeMechanisms`, steg 8 = `answerUnique`, steg 9 =
  `skepticAssessment` (stoppregeln kräver inget anrop), steg 10 =
  `closingSummary` + `closingAssessment` + `findInsight`; +1 vid
  följdfråga i steg 2, 3, 4, 6 och 8. Utan följdfrågor och rättelser
  blir det omkring 17 AI-anrop per session.
- Varje anrop efter steg 1 får hela kontexten (`contextBlock`) och
  stegets fråga och svar (`qaBlock`): personprofilen, tidigare frågor
  och svar ordagrant (märkta "(valt förslag)" när svaret var ett klickat
  förslag), idékartans rättelser, insikterna med användarens reaktion
  och efter hand problemkartan, ramen, "Det här hittade vi ute i
  världen", kärnan, det unika och affären, sakerna värda att testa och
  slutkortet — med instruktionen att bygga vidare på dem och aldrig
  fråga om något som redan är besvarat.

### 0.4 Svara genom att klicka eller skriva

- 2–4 klickbara förslag i jag-form (högst 12 ord), byggda ENBART på det
  användaren redan skrivit eller valt, i olika vinklar och utan
  påhittade siffror, namn, årtal, titlar eller erfarenheter. Visas som
  vägval-rader (inga pill-chips) under rubriken "Klicka, ändra eller
  skriv fritt — korta svar räcker". Ett klick lägger förslaget som en
  rad i textrutan, där det kan ändras.
- Svarets källa sparas i `ctx.answers[].source`: `eget`, `forslag` eller
  `blandat`. Ett klickat förslag är Boggels text ordagrant och räknas
  aldrig som "dina ord" (0.8).
- "Skriv själv först" (förslagen dolda tills användaren skrivit något
  eller klickat "Jag vet inte – visa förslag") gäller BARA steg 1, 5 och
  7. I steg 1 är det skrivstartare ("Kör du fast? Börja en mening så
  här": t.ex. "Vem vill du hjälpa?"), inte AI-förslag — Boggel hittar
  fortfarande inte på idéer — och de visas bara när användaren ber om
  dem, så att de inte blir en checklista bredvid huvudfrågan. I steg 5
  och 7 har frågan inga förslag (`SkarpOpenQuestion`): AI:ns alternativ
  hämtas först när användaren skrivit sin egen formulering — de ska då
  vara verkliga alternativ till den, inte omskrivningar — eller klickat
  "Jag vet inte – visa förslag" (anropet utan egen text). Länken syns
  bara medan textrutan är tom, så ett klick kastar aldrig bort det
  användaren skrivit. Övriga steg visar förslagen direkt: 2, 3, 4, 6, 8
  och stoppregeln i 9. Ersätter §1 punkt 2:s och §8.2:s fördelning av
  "Du först".
- **Valskärmen i steg 5 och 7** (`ChoiceList`): rubriken "Så kan
  problemet ramas in." med raden "Varje ram bygger på det du skrev — ju
  fler av dina ord, desto närmare dig." (steg 7: "Så kan kärnan se ut."
  och "Varje väg bygger på …"). Överst "BYGGER PÅ DINA ORD": de
  ordagranna citat ur användarens texter som alternativen bygger på,
  numrerade med romber (blocket utelämnas om det inte finns några).
  Sedan en rad per alternativ:
  användarens egen formulering först ("DIN EGEN", "Dina ord"; förvald
  när den finns), därefter A–C med typetikett (ramtyp/tankeoperation i
  vardagsord, `labels.ts`), rubriken (högst 10 ord), citatnumren och
  närheten — tre romber + "Nära dig" (2–3 citat), "En bit bort" (1)
  eller "Längre bort" (0). Närheten räknas i kod av hur många citat som
  verifierats i användarens texter, aldrig av AI:n. Ett valt alternativ
  får sina citat guldmarkerade (övriga dämpas) och öppnar under raden:
  varför det passar, "TALAR FÖR", "TÄNK PÅ" och citaten. Åtgärder:
  "Välj den här →" (egen: "Behåll min →"), "Skriv om med egna ord"
  (förifylld text att justera, "Använd min version →") och "Titta på de
  andra".
- Valet görs utan nytt AI-anrop. Varje alternativ har med sig sin egen
  spegling och kartrörelse, och den egna formuleringen sin (`own`), så
  romben flyttar sig efter det faktiska valet (avgör §8.3); typetiketten
  står i en egen rad, inte i texten (avgör §8.5). Ett AI-alternativ blir
  `ctx.frame`/`ctx.mechanism` med ursprung `ai_tolkning` och sparas
  INTE i `ctx.answers`; en egen eller omskriven formulering blir
  `anvandare` och sparas i `ctx.answers` (`eget` resp. `blandat`). Egen
  formulering utan `own` ger ingen spegling och ingen rörelse.

### 0.5 Välkomnande och spegling

- **Steg 1, "Det här hörde jag":** välkomnandet är ett stycke på högst
  två meningar överst: `readIdea`s `welcome` — EN mening (högst 30 ord)
  som visar att Boggel förstått vad som bär idén, utan hälsningsfras och
  beröm — följd av den fasta meningen från `welcomeTimeSentence()` ("Vi
  tar tio korta steg, ungefär 15 minuter, så har du en tydlig bild av
  idén och vad du ska testa först — något att ta med till din
  rådgivare."). Idétexten visas på linjerat papper där 2–5 noteringar
  avslöjas en i taget: citatet guldmarkeras och numreras, och raden
  säger DU VET / DU TROR / DU HOPPAS (`fakta`/`tolkning`/`hopp`) plus
  vad det betyder. Sist "Kort sagt: {ideaShort}" — det som hamnar i
  idékartan. Användaren kan markera en notering "stämmer inte" (den
  faller bort ur `ctx.mirror` men sparas inte som ett svar — där står
  bara användarens egna ord), gå vidare ("Stämmer, vidare →") eller
  rätta ("Inte riktigt — jag menar …"), vilket kör `readIdea` igen med
  rättelsen.
- **Steg 2–9:** efter varje svar en spegling på EN mening (`reflection`,
  högst 25 ord) — ett påstående med användarens egna ord
  (`reflectionQuote`, 2–6 ord, kontrolleras i kod), aldrig en fråga,
  aldrig beröm, aldrig inledd med "Du säger", "Du skrev" eller "Det
  låter". Den blir lead på nästa skärm och står, med kartrörelsen i ord,
  i notisraden under kartan; idékartan fylls synligt. I steg 5 och 7 är
  det det valda alternativets spegling (eller `own.reflection`).
  Stoppregeln speglas av `closingSummary` (`stopRuleReflection`), som
  lead på slutkortet. Efter en insiktsskärm (före steg 4 och 7) har
  nästa skärm ingen lead — speglingen stod redan över insikten.
- `ctx.mirror` byggs fortfarande av noteringarna (per `kind`), så steg
  4–10 får samma underlag som i fas 2.

### 0.6 Insiktsögonblick

- Tre ögonblick (`InsightMoment`): `efter-kunden` (efter steg 3),
  `efter-verkligheten` (efter steg 6) och `slutkort` (i slutkortet).
- `findInsight` (Syntetiseraren, starkare modellnivå) hittar EN konkret
  koppling, motsägelse eller ett mönster i det användaren skrivit, helst
  mellan olika steg: 1–2 meningar (högst 45 ord) om vad det betyder för
  idén, med användarens ord inom citattecken. Aldrig allmänna råd som
  skulle passa vilken idé som helst ("prata med kunder", "validera
  tidigt"). Efter steg 6 kopplar den användarens ord till det som
  hittades ute i världen (t.ex. att ett alternativ redan tar betalt,
  eller att det användaren beskrivit som unikt redan finns); i
  slutkortet lyfter den det viktigaste mönstret eller den viktigaste
  motsägelsen i hela sessionen, som användaren troligen inte sett.
- Citaten (1–2) måste stå ordagrant i användarens egna texter — idén,
  svaren och idékartans rättelser, från vilket av de nio stegen som
  helst. Ett klickat förslag och Boggels egna texter räknas aldrig.
  `finding` är namnet på det alternativ insikten bygger på, ordagrant ur
  "Det här hittade vi ute i världen"; det matchas mot `ctx.evidenceMap`
  enligt namnregeln (0.10) och får då sin länk, annars null (alltid null
  efter steg 3).
- Efter steg 3 och 6 visas insikten som en egen skärm (`InsightCard`,
  kicker "Steg 3 av 10"/"Steg 6 av 10" + "En koppling i det du
  skrivit", "Två saker som drar isär" eller "Ett mönster"): insikten
  stort, citaten den bygger på med stegetikett, efter steg 6 ev. en rad
  "Hittat på nätet" med länk, och raden "Det här kan du ta med till din
  rådgivare." I slutkortet står den direkt efter "Det som är starkt"
  (0.11). Användaren svarar "Stämmer" eller "Inte riktigt" (med valfri
  rättelse); reaktionen sparas i `ctx.insights` och följer med i alla
  senare anrop.
- `findInsight` startas parallellt med stegets eget anrop
  (`answerCustomer`, `answerReality`, slutkortets två anrop) så att
  väntan blir kort; ett nytt försök med samma svar återanvänder samma
  anrop. Hittas ingen
  insikt (eller anropet fallerar) är det inget fel: efter steg 3 och 6
  en lugn rad ("hellre ingen än en gissning") med "Vidare till steg 4 →"
  resp. "Vidare till steg 7 →" och "Leta en gång till"; i slutkortet
  utelämnas den. Insikten blockerar aldrig flödet.

### 0.7 Ton och mjuk kontroll

- `BOGGELS_ROST` (`roles.ts`) monteras efter rollen i alla stegens anrop
  (0.12) och går före rollbeskrivningen där de krockar: du-tilltal,
  korta meningar, inga utropstecken eller emojis, utgå alltid från något
  användaren skrivit, visa kunskap genom att sätta ord på det som bär
  eller skaver i just den här idén. Be aldrig om ett exempel bara för
  att ett svar är kort (Intervjuarens äldre regel) och fråga aldrig om
  Boggel förstått rätt (Spegelns). Förbjudet
  bl.a.: "Det är en intressant idé", "Bra fråga", "Låt oss utforska",
  "Det finns många faktorer att tänka på", "Vilken spännande …", "Det
  låter som …". Inget beröm.
- `SKARP_BASE`s ton: varm, kunnig, konkret — som en erfaren
  affärsrådgivare. Boggel ersätter inte användarens rådgivare; den
  hjälper användaren att tänka klarare och komma förberedd.
- Promptexemplen gäller medvetet en ANNAN idé (pensionerade hantverkare
  som lär ut reparationer) så att modellen kopierar formen, inte
  innehållet.
- Hitta aldrig på siffror, marknadsstorlekar, namn eller källor. Namn på
  alternativ och konkurrenter får BARA komma från research-underlaget
  (0.10).
- `generateSkarp` (`validators.ts`): ovanpå den hårda valideringen
  (`validateNoPlaceholders`) kontrolleras längd, en mening, frågetecken
  och förbjudna fraser (på allt användaren ser); vid problem körs EN
  rättande runda. Efter sista rundan används svaret ändå och
  efterbearbetas (trimmas till hela meningar, citat och namn som inte
  hittas nollas) — mock-läget och en envis modell blockerar aldrig
  flödet.

### 0.8 Evidensmärkning

- Etiketterna är synliga men diskreta (`ORIGIN_LABELS` i `render.ts`),
  avgör §8.1:
  - "Dina ord" (`anvandare`): citaten i idékartan, en egen eller
    omskriven ram/kärna, citaten bakom alternativen i steg 5/7, det
    starka i slutkortet och stoppregeln när användaren skrivit den
    själv (ett klickat förslag utan ändring är "Boggels tolkning").
  - "Boggels tolkning" (`ai_tolkning`): idékartans korta formuleringar
    och en vald AI-ram/-kärna, vad alternativen betyder (`demandNote`),
    "Om kunden inte gör något", "Det vi inte vet än", "Saker värda att
    testa", idén i en mening, "Så står idén nu" och "Ditt första test".
  - "Hittat på nätet" (`extern_kalla`): alternativen i steg 6, och de
    alternativ som en insikt, en sak värd att testa eller varningen från
    steg 8 nämner — länken kommer alltid från research-fyndet.
- Citat som visas som användarens ord (noteringar, `*Quote`-fälten,
  insiktens citat, alternativens citat, det starka i slutkortet,
  Skeptikerns `quote`) kontrolleras i kod: de behålls bara om de står
  ordagrant i användarens text (`findQuoteSpan`/`findQuoteInSources` i
  `text.ts` — tål skiftläge, blanksteg och citattecken, returnerar
  originalets exakta delsträng). Annars visas ingen markering.
- "Dina ord" är BARA text användaren själv skrivit: idén, svaren och
  idékartans rättelser (`userSources` i `choice.ts`). Ett klickat
  förslag (`source: "forslag"`) räknas aldrig, inte heller i insikterna.

### 0.9 Steg 4–10, steg för steg

Samma konstruktion som steg 1–3 (0.3–0.5). "Lead" är speglingen överst
på skärmen (0.5); `"…"` i väntetexterna är början av användarens svar.

#### Steg 4 — Utvidga problemet

- **Fråga:** `answerCustomer`s `nextQuestion` — den vinkel som saknas
  (varför det är ett problem, vad som orsakar det, vem mer som påverkas,
  vad som händer före/efter, vilket större mål som ligger bakom). Ingen
  lead — insikten var förra skärmen.
- **Vem går först:** användaren, med förslagen synliga direkt.
- **Anrop:** `answerProblem` (+1 vid följdfråga). Väntan: `Läser "…"`,
  "Ser vad problemet hänger ihop med …", "Förbereder nästa fråga …".
  Anropet skriver också steg 5:s fråga.
- **Sparas:** `ctx.problemMap` (orsaker, fler berörda, konsekvenser,
  större mål — bara det som framgår, tomma listor är ok) och svaret som
  `problemet`.
- **Idékartan:** Problemet ← `problemShort` (högst 15 ord).

#### Steg 5 — Välj problemram

- **Fråga:** `frameQuestion`, med `answerProblem`s spegling som lead. Den
  ber om en OMFORMULERING av samma problem utan lösningen (inte en ny
  beskrivning — användaren har redan beskrivit det två gånger) och
  nämner lösningen som ska lämnas utanför med användarens ord. För en
  annan idé: "Hur låter problemet om kursen inte fanns?"
- **Vem går först:** användaren (skriv själv först, 0.4).
- **Anrop:** `reframeProblem`, med eller utan `ownFrame`: 2–3
  lösningsneutrala ramar (ingen teknik, produkt eller affärsmodell) av
  typen orsak, aktör, system, abstraktion eller motsats ("Orsaken
  bakom", "Från någon annans håll", "Hela kedjan", "Ett steg upp",
  "Tvärtom"), var och en med varför den passar, talar för, tänk på (ett
  ärligt antagande), citat, spegling och kartrörelse. Väntan: `Läser
  "…"`, "Letar andra sätt att se samma problem …".
- **Val:** valskärmen (0.4), lead = `own.reflection` om den finns, den
  egna ramen förvald.
- **Sparas:** en egen ram sparas som `problemram` (`eget`) redan när den
  skickas, före valskärmen. `ctx.frame` och en omskrivning (som
  `problemram`, `blandat`) sparas först när steg 6 är klart att visa.
  Misslyckas steg 6 sparas inget av det, kartan står still och valet
  kan göras om, så inget sparas eller flyttas två gånger.
- **Idékartan:** Problemramen ← den valda ramen.

#### Steg 6 — Verkligheten

- **Skärm:** lead = den valda ramens spegling (ingen när den egna ramen
  behölls — den speglingen var redan valskärmens lead), rubriken "Det här hittade
  vi ute i världen." (ingen why-rad) och alternativen (0.10), sedan
  stegets fråga i `QuestionBlock` med förslag.
- **Vem går först:** AI:n (researchen), sedan användaren.
- **Anrop:** `buildReality` + `mapCompetitors` (oförändrad) parallellt.
  Väntan: "Kollar vad som redan finns där ute …", "Sorterar det vi
  hittade …", "Ser vad det betyder för din idé …". `buildReality` ger
  också stegets fråga, som utgår från det som hittades och användarens
  egna ord (vad som skiljer sig, vad som överraskar, vad kunden gör i
  dag). Vid svar: `answerReality` (+1 vid följdfråga) och parallellt
  `findInsight("efter-verkligheten")`. Väntan: `Läser "…"`, "Letar efter
  något du kanske inte sett …". `answerReality` skriver också steg 7:s
  fråga.
- **Sparas:** `ctx.evidenceMap` DIREKT när `buildReality` kommer tillbaka
  (alternativen med `what` och `sourceUrl`, `doNothing`, `gaps`,
  `demandNote`, `reaction: ""`), så att `answerReality` och insikten ser
  alternativen; när svaret är klart blir det `reaction` och sparas som
  `verkligheten`. Konkurrentkartan som förut (§6).
- **Idékartan:** Verkligheten ← `realityShort` (vad verkligheten säger
  om idén, högst 18 ord, knutet till det som hittades).
- **Sedan:** insikt 2 (0.6), med `answerReality`s spegling som lead.

#### Steg 7 — Lösningskärnan

- **Fråga:** `mechanismQuestion` — "om du tar bort alla funktioner, vad
  är kärnan i hur din lösning fungerar?" anpassad till idén. Ingen lead
  — insikten var förra skärmen.
- **Vem går först:** användaren (skriv själv först, 0.4).
- **Anrop:** `proposeMechanisms`, med eller utan `ownMechanism`: 2–3
  kärnor, var och en en namngiven tankeoperation ("Ta bort ett steg",
  "Byt betalare", "Gör det manuellt först", "Förebygg i stället för att
  laga", "Hämta från en annan bransch", "Utgå från ett extremfall") med
  en rubrik som är en princip för HUR det fungerar — aldrig en
  funktionslista eller en färdig produkt — och samma fält som i steg 5.
  Väntan: `Läser "…"`, "Prövar andra sätt att få det att fungera …".
  Anropet skriver också steg 8:s fråga: det som saknas MEST av vad som
  gör kombinationen ovanlig och värdefull, vem som betalar och första
  vägen till kunderna. Den förutsätter inte vilket alternativ som väljs,
  och förslagen bygger på personprofilen, svaren och det som hittades —
  aldrig "ingen annan gör det".
- **Val:** valskärmen (0.4), lead = `own.reflection` om den finns, den
  egna kärnan förvald.
- **Sparas:** `ctx.mechanism` (+ svaret som `mekanism` när kärnan är
  egen eller omskriven).
- **Idékartan:** Kärnan ← den valda kärnan.

#### Steg 8 — Det unika och affären

- **Fråga:** `proposeMechanisms`s `nextQuestion`, med den valda kärnans
  spegling som lead (ingen när den egna kärnan behölls — den speglingen
  var redan valskärmens lead).
- **Vem går först:** användaren, med förslagen synliga direkt.
- **Anrop:** `answerUnique` (+1 vid följdfråga). Väntan: `Läser "…"`,
  "Jämför med det vi hittade ute i världen …". Unikt = en ovanlig,
  värdefull KOMBINATION, inte "ingen annan gör det" (§1 punkt 5).
- **Sparas:** `ctx.uniqueAndBusiness`: `uniqueCombination` (en–två
  meningar med användarens ord), `conflictWarning` och `anvandare`,
  `kopare`, `betalare`, `forstaKanal` — bara det svaren och underlaget
  säger, annars null (aldrig "okänt"). `conflictWarning` är EN mening om
  att något användaren pekar ut som unikt redan görs av ett alternativ
  ute i världen; den behålls bara om den nämner ett alternativ enligt
  namnregeln (0.10) och får då dess länk (`conflictFinding`). Svaret
  sparas som `unikt-affar`. Allt det här sparas först när steg 9 är
  klart att visa; misslyckas steg 9 återanvänds svaret på steg 8 (inget
  nytt anrop) när användaren försöker igen.
- **Idékartan:** Det unika och affären ← `uniqueShort` (högst 15 ord).

#### Steg 9 — Skeptikern

- **Skärm:** lead = `answerUnique`s spegling, rubriken "Saker värda att
  testa." med raden "Det här är det som avgör om idén håller — inget av
  det är ett nej.", ev. varningen från steg 8, punkterna
  (`TestList`) och sist stoppregeln i `QuestionBlock` med förslag.
- **Vem går först:** AI:n (Skeptikern), sedan användaren med
  stoppregeln.
- **Anrop:** `skepticAssessment` — fortfarande ett eget, isolerat anrop.
  Väntan: "Skeptikern läser igenom allt du skrivit …", "Väljer det som
  är mest värt att testa …". 1–3 punkter av de viktigaste typerna ("Det
  som måste stämma", "Det som talar emot", "Varför ingen redan löst
  det", "Lätt att kopiera", "Behövs tekniken?"): en rak mening om vad
  som MÅSTE stämma eller talar emot, med användarens ord där det går
  (`quote`, kontrollerat i kod), och EN mening om det enklaste,
  billigaste sättet att ta reda på det inom några dagar — ett test,
  ingen lösning, inga påhittade siffror. Nämner en punkt ett alternativ
  ute i världen får den dess länk. Ingen positiv sammanfattning.
  `stopQuestion` ("Vad skulle få dig att byta riktning eller lägga ner?"
  anpassad till idén) har förslag byggda på punkterna, utan siffror —
  tröskeln lämnas till användaren. Stoppregeln kräver inget eget anrop.
- **Sparas:** `ctx.skeptic` (`concerns` med typ, punkt, test och citat;
  `stopRule` och `stopRuleSource` — "forslag" när ett förslag klickats
  utan ändring, och då kallas orden aldrig användarens egna) och
  stoppregeln som `skeptikern`.
- **Idékartan:** Att testa ← den första saken värd att testa och
  "Stoppregel: …", direkt när stoppregeln är skriven.

#### Steg 10 — Slutkort och test

- **Skärm:** lead = `stopRuleReflection`, rubriken "Här har du hamnat.",
  slutkortet (0.11) och följdfrågorna; kartan zoomas ut.
- **Vem går först:** AI:n (Syntetiseraren), enbart från insamlat
  material.
- **Anrop:** `closingSummary` + `closingAssessment` +
  `findInsight("slutkort")` parallellt; kortet visas så fort alla tre är
  klara, och `finishSkarpSession` sparar det sedan en gång i
  bakgrunden. Väntan: "Samlar ihop det som är starkt …", "Ser var idén
  står i dag …", "Skriver frågor till din rådgivare …".
- **Sparas:** `ctx.closing` (idén i en mening, områdena, första testets
  hypotes), så att följdfrågorna vet vad kortet sa, och slutkortets text
  (0.11).

### 0.10 Det här hittade vi ute i världen

- Researchen körs oförändrad en gång per session, i bakgrunden direkt
  efter steg 3 (`triggerCompetitorResearch`, ämnet `konkurrenter`,
  frågan från `buildCompetitorQuery(idé, situationsmening)`), så att
  steg 4–5 ger den tid. Rättar användaren "Kunden och situationen" i
  idékartan före steg 6 får researchen sitt enda omtag (högst två
  körningar totalt, som tidigare vid "Redigera kunden"). Därefter byts
  situationsmeningen — som också är research-frågan — inte längre, så
  steg 6 träffar cachen; rättelsen når ändå alla anrop via `ctx.ideaMap`.
- `buildReality` läser resultatet med EXAKT samma fråga (cacheträff).
  Det används sedan i steg 6, 8 och 9, i insikterna och i slutkortet,
  via kontextblocket "DET HÄR HITTADE VI UTE I VÄRLDEN" — de enda namn
  AI:n får nämna.
- `RealityView` visar högst 6 alternativ, de direkta först, som stiger
  in ett i taget. Varje rad: namnet ordagrant ur underlaget, "SAMMA SAK" (direkt: samma
  problem, liknande sätt) eller "ANNAT SÄTT" (indirekt: annat sätt eller
  näraliggande problem), EN mening om vad det är eller gör — bara från
  underlaget, inga siffror som inte står där — och länken "Hittat på
  nätet ↗". Länken kommer alltid från research-fyndet, aldrig från AI:n;
  ett alternativ vars namn inte matchar något fynd räknas som påhittat
  och tas bort. Dubbletter tas bort på normaliserat namn (flera
  alternativ får dela länk).
- Ett namn AI:n skrivit i någon runda men som inte visas (påhittat,
  borttaget i rättelserundan eller utanför de sex) får inte stå kvar i
  frågan, förslagen, `demandNote`, luckorna eller `doNothing` — utom
  när användaren själv skrivit namnet. Rättelserundan ber om det; står
  det ändå kvar tas förslaget, luckan eller raden bort, och nämner
  frågan namnet (eller finns fynd men inget alternativ kvar) blir den
  neutral: "Vad gör kunden i dag i stället?".
- Finns direkta alternativ följer EN mening om vad det betyder
  (`demandNote`, "Boggels tolkning"): att andra redan tar betalt för att
  lösa det kan betyda att det finns efterfrågan, och vad som då skiljer
  användarens idé. Därefter "OM KUNDEN INTE GÖR NÅGOT" (`doNothing`, med
  användarens ord om dagens lösning) och "DET VI INTE VET ÄN" (0–3
  luckor), båda "Boggels tolkning".
- Hittas inget körs anropet ändå: inga alternativ, en lugn rad ("Vi
  hittade inget på nätet som liknar din idé. Det kan betyda en lucka —
  eller att kunderna löser det på ett annat sätt.") och en fråga om vad
  kunden gör i dag.
- **Namnregeln** (insikternas `finding`, steg 8:s varning, steg 9:s
  punkter): båda namnen normaliseras (`normalizeForMatch`), namn kortare
  än 3 tecken ignoreras, och namnet måste stå som hela ord — aldrig som
  ren delsträng ("Go" matchar inte "goda"). Tomt namn = ingen träff.
  Utan träff ingen länk, och i steg 8 ingen varning.

### 0.11 Slutkortet

Inga sifferbetyg, staplar eller procent. `ClosingCard` visar, i den här
ordningen (en del utan innehåll utelämnas):

1. **"Här har du hamnat."** — idén i EN mening som går att säga högt
   (`ideaOneLiner`, "Boggels tolkning"), med användarens ord
   guldmarkerade om de står i meningen.
2. **Det som är starkt** — 1–3 saker som är starka i idén: fakta om idén
   som användarens egna ord visar, aldrig beröm av personen, var och en
   med ett ordagrant citat ("Dina ord"). En styrka utan verifierat citat
   tas bort; blir listan tom står det första området med nivån Starkt
   eller På god väg här i stället.
3. **Insikten** (0.6), om en hittats — belöningen, direkt efter det
   starka, med "Stämmer"/"Inte riktigt" och "Noterat." när användaren
   svarat.
4. **Så står idén nu** ("Boggels tolkning") — de sex områdena (Problemet,
   Du som grundare, Det som skiljer lösningen, Genomförbarhet, Affären,
   Hur snabbt du kan testa), det starka först, var och en med en nivå:
   **Starkt** (bara när underlaget tydligt visar det), **På god väg**
   eller **Behöver mer underlag** (mest antaganden), och EN mening om
   nästa steg som nämner något konkret ur användarens ord och inte
   upprepar en sak värd att testa. Vad bedömningen bygger på visas bara
   vid "Behöver mer underlag".
5. **Din idé, del för del** — idékartans fyllda fält i stegordning med
   ursprung, sist "Använder · Köper · Betalar · Första kanal" (det som
   saknas: "Inte klart än").
6. **Saker värda att testa** ("Boggels tolkning") — Skeptikerns punkter
   med test och ev. länk, ev. varningen från steg 8 och "DIN STOPPREGEL"
   ("Dina ord" när användaren skrivit den, "Boggels tolkning" när ett
   förslag klickats utan ändring).
7. **Ditt första test** ("Boggels tolkning") — den FÖRSTA saken värd att
   testa utvecklad till ett konkret test, inget nytt: Hypotes, Så testar
   du, Det räknas som bevis (knutet till stoppregeln om den finns), Inom
   48 timmar.
8. **Det här kan du ta med till din rådgivare** — exakt tre frågor,
   grundade i det som fortfarande är osäkert.
9. **"Kopiera slutkortet"** och "Börja om". Kopian är markdown som
   byggs i klienten (`renderClosingCardMarkdown`, en ren funktion) av
   det som visas, inklusive användarens svar på insikten.
   `finishSkarpSession` sparar texten med `saveSession` (`resultText`,
   oförändrad signatur) när det finns ett klient-id — en gång, när kortet
   först visas, så den sparade texten saknar svaret på insikten (det
   finns i kopian och i `ctx.insights`).

Under kortet: **Fråga Boggel om idén** (`FollowupDialog`,
`askSkarpFollowup` oförändrad) — följdfrågor och invändningar som Boggel
svarar kort på. Bara den senaste frågan och svaret syns; tidigare ligger
hopfällda under "Tidigare frågor (n)". Ingen chattlogg.

Avgör §8.6 (steg 10 är två anrop — sammanfattning och bedömning — plus
insikten) och §8.7 (slutkortet visas i `ClosingCard`, inte i
`ResultReport`, som är Kompass-lägets och orörd).

### 0.12 Anrop och scheman — var de bor

Alla i repot `gxy652fxr`, under `src/lib/skarp/`. Prompten monteras
likadant överallt: `SKARP_BASE` + roll + `BOGGELS_ROST` (steg 6 och 8
har ingen roll).

| Anrop | Fil | `schemaName` / schema | Roll |
| --- | --- | --- | --- |
| `readIdea` | `step1-idea.ts` | `skarp_read_idea` / `ReadIdeaSchema` | Spegeln |
| `answerWhyYou` | `step2-why.ts` | `skarp_why_you` / `WhyYouResultSchema` | Intervjuaren |
| `answerCustomer` | `step3-customer.ts` | `skarp_customer` / `CustomerResultSchema` | Intervjuaren |
| `answerProblem` | `step4-problem.ts` | `skarp_problem` / `ProblemResultSchema` | Intervjuaren |
| `reframeProblem` | `step5-reframe.ts` | `skarp_reframe` / `ReframeResultSchema` | Omformuleraren\* |
| `buildReality` | `step6-reality.ts` | `skarp_reality` / `RealityResultSchema` | – |
| `answerReality` | `step6-reality.ts` | `skarp_reality_answer` / `RealityAnswerSchema` | – |
| `proposeMechanisms` | `step7-mechanism.ts` | `skarp_mechanism` / `MechanismResultSchema` | Linsen |
| `answerUnique` | `step8-unique.ts` | `skarp_unique` / `UniqueResultSchema` | – |
| `skepticAssessment` | `step9-skeptic.ts` | `skarp_skeptic` / `SkepticResultSchema` | Skeptikern\* |
| `closingSummary` | `step10-closing.ts` | `skarp_closing_summary` / `ClosingSummarySchema` | Syntetiseraren\* |
| `closingAssessment` | `step10-closing.ts` | `skarp_closing_assessment` / `ClosingAssessmentSchema` | Syntetiseraren\* |
| `findInsight` | `insight.ts` | `skarp_insight` / `InsightSchema` | Syntetiseraren\* |

\* Starkare modellnivå (`modelTier: "strong"`).

- Oförändrade: `initSkarpMap` (kartans axlar), `triggerCompetitorResearch`
  (`research.ts`, inget AI-anrop), `mapCompetitors` (`step6-reality.ts`,
  konkurrentkartan) och `askSkarpFollowup` (`step10-closing.ts`,
  följdfrågorna efter slutkortet). `finishSkarpSession`
  (`step10-closing.ts`) är inget AI-anrop; den sparar slutkortets text.
- Varje fil exporterar en testbar `*Logic`-funktion och en
  `createServerFn`-wrapper; allt exporteras via `index.ts`. Indata:
  svarsstegen (2, 3, 4, 6-svaret, 8) tar `{ context, main, followUp? }`;
  `reframeProblem`/`proposeMechanisms` tar `{ context, ownFrame? }`
  resp. `{ context, ownMechanism? }`; `buildReality`,
  `skepticAssessment`, `closingSummary` och `closingAssessment` tar
  `{ context }`; `findInsight` tar `{ context, moment }`.
- Borttaget i fas 3: `mirrorIdea`, `planWhyYouQuestions`,
  `synthesizePersonprofil`, `planCustomerQuestions`,
  `synthesizeSituation`, `planProblemQuestions`, `synthesizeProblemMap`,
  `buildEvidenceMap`, `checkUniqueAndBusiness`,
  `synthesizeClosingCardPart1`/`Part2`, `KEEP_OWN_FRAME_MOVE`,
  `KEEP_OWN_MECHANISM_MOVE`, `reframeToSuggestion`,
  `mechanismToSuggestion`, de gamla siffer- och säkerhetsetiketterna i
  `render.ts`, filerna `interview.ts` och `suggestions.ts` och
  komponenten `EvidenceCard`. `reframeProblem`, `proposeMechanisms` och
  `skepticAssessment` har kvar sina namn men är omskrivna.
- Scheman: `schemas.ts` (Skärp-ägda; `src/lib/ai/schemas.ts` är orörd).
  Visad prosa trunkeras aldrig i schemat — korthet styrs av prompt, mjuk
  kontroll och efterbearbetning.
- Steg 3: situationens sju slotar är `string | null` (en del som inte
  går att utläsa blir null). `renderSituationSentence` bygger
  mallmeningen när alla sju finns, annars visas `kundShort`.
- Kontext (`SkarpContextSchema`): `answers` (steg-id:n `ide`,
  `varfor-du`, `kunden`, `problemet`, `problemram`, `verkligheten`,
  `mekanism`, `unikt-affar`, `skeptikern`), `ideaMap`, `insights`, och
  för steg 4–10 `problemMap`, `frame`, `evidenceMap` (med `what`,
  `sourceUrl` och `demandNote`), `mechanism`, `uniqueAndBusiness`,
  `skeptic` (`concerns` med test och citat) och `closing`.
- Hjälpare: `text.ts` (citatmatchning, meningsdelning och -trimning),
  `validators.ts` (`generateSkarp`, `cleanQuestion`, `settleFollowUp`,
  `verifiedQuote`, förbjudna fraser), `choice.ts` (`settleChoiceOptions`,
  `settleOwnChoice`, `userSources`), `labels.ts` (svenska etiketter för
  ramtyper, tankeoperationer, Skeptikerns typer, områden och nivåer —
  alltid i kod, aldrig av AI:n), `render.ts` (`renderSituationSentence`,
  `ORIGIN_LABELS`, `renderClosingCardMarkdown`), `progress.ts` (tid
  kvar), `map-note.ts` (kartrörelsen i ord), `roles.ts` (`BOGGELS_ROST`).
- UI: `src/components/skarp/` (`SkarpProgress`, `StepScreen`,
  `AnswerComposer`, `IdeaReadView`, `InsightCard`, `IdeaMap`,
  `ThinkingLine`, `FocusError`, `SkarpRail`, och för steg 4–10
  `ChoiceList`, `RealityView`, `QuestionBlock`, `TestList`,
  `ClosingCard`, `FollowupDialog`), delade primitiver enligt prototypen
  i `src/components/shared/boggel-ui.tsx` (`PrimaryButton`,
  `TextButton`, `Kicker`, `NumberRomb`, `Highlight`, `GoldDash`,
  `LinedPaper`) och guldtonerna/markeringarna i `src/styles.css`.
  Visuellt facit: `design-reference/skarp/prototyp.html`.

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

**Historik (fas 2) — helt ersatt av §0.3.** Inget steg planerar längre
flera frågor eller sammanställer dem i ett andra anrop: varje steg har en
huvudfråga från föregående stegs anrop och högst en följdfråga.

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

**Historik (fas 2) — helt ersatt av §0.** Står kvar för att visa
varifrån stegens syfte och ordning kommer; frågor, scheman, anrop och
skärmar nedan gäller inte längre (steg 1–3: §0.1–0.8, steg 4–10: §0.9–0.11,
anropen: §0.12).

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

### 8.11 Pre-existerande dokument-inkonsekvenser — ÅTGÄRDADE i fas 2

Båda flaggades här i fas 1 och är bekräftade av dig (svar punkt 11,
"ja, fixa dem — rent dokumentation"):

- `product.md`/`architecture.md`/`roadmap.md`/`decisions.md` låg i
  repots ROT trots att CLAUDE.md pekar på `docs/`-prefixade paths — en
  äldre path-förskjutning, inte orsakad av detta arbete. Flyttade (med
  `git mv`, historiken bevarad) till `docs/product.md`,
  `docs/architecture.md`, `docs/roadmap.md`, `docs/decisions.md`. Alla
  interna korsreferenser mellan dokumenten (inklusive i det här
  dokumentet och i `design-reference/HANDOFF.md`) är uppdaterade till
  de nya paths.
- `architecture.md` beskrev kärn-AI:t som "Claude API", men den
  faktiska implementationen (sedan tidigare arbete) kör OpenRouter/
  Gemma via ett OpenAI-kompatibelt lager (`AI_PROVIDER=openai-compatible`,
  se README/.env.example i gxy652fxr), med Anthropics Claude API som
  ett alternativt, färdigkopplat men ej aktiverat val i samma
  abstraktion. Omskrivet i `docs/architecture.md` för att beskriva det
  leverantörsoberoende lagret korrekt; samma stale beskrivning fanns
  även i `docs/decisions.md` ("Två separata AI-roller") och är rättad
  där också.

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
