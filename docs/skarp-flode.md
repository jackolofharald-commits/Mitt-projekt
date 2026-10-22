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
Ovanpå fas 3 ligger Skärpan (§0.13): en kunskapsbas av tankeverktyg,
klassificeringen efter steg 1, det deterministiska verktygsvalet och den
nya rösten. **§0 gäller.** §1–§10 beskriver fas 1–2 och står kvar som
historik; där de krockar med §0 gäller §0.

## 0. Fas 3 — upplevelse och konstruktion (alla tio steg ombyggda)

Fas 2 fungerade tekniskt men upplevdes som en promptad AI-chatt och ett
förhör. Fas 3 bygger om upplevelse, konstruktion, ton och design i alla
tio steg; stegens syfte och ordning (§7) ligger kvar. Steg 1–3 byggdes
först som provexemplar, med välkomnandet, idékartan och det första
insiktsögonblicket. Steg 4–10 är, efter användarens godkännande,
ombyggda på samma sätt, med insikter även efter steg 6 och i slutkortet.
Skärpan (0.13) gör sedan frågorna skarpare för grundare som redan kan
grunderna — utan att röra stegen, skärmarna, designen eller
ärlighetsreglerna.

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
  "· Boggel läser …" i kickern, på en följdfråga "· om du vill" (0.3),
  och på en huvudfråga som tar sitt perspektiv från ett tankeverktyg
  dess etikett (t.ex. "STEG 3 AV 10 · KUNDEN I SITUATIONEN · VEM SOM
  SKRIVER UNDER", 0.13).
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

- Varje steg har EN huvudfråga: EN mening (högst 16 ord och 130 tecken,
  ett "?") med ankaret i användarens egna ord i samma mening, plus en
  kort rad om vad svaret AVGÖR för just den här idén (`why`, en mening,
  högst 16 ord och 130 tecken — aldrig "det hjälper oss att förstå
  dig"). Frågan väljs efter informationsvärde inom stegets frågefält
  (0.13): det mest osäkra och avgörande, gärna i det senaste svaret, och
  aldrig något som redan står i idén eller svaren. Den väljs aldrig ur en
  fast lista och genereras i föregående stegs anrop:

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
  samma skärm (`QuestionBlock`, kicker "DIN TUR", med perspektivet
  "DIN TUR · …" när frågan har ett, 0.13).
- Högst en följdfråga, och bara när svaret inte säger NÅGOT om stegets
  kärna (steg 2: varför just du; steg 3: vem och i vilken situation;
  steg 4: problemet; steg 6: verkligheten; steg 8: det huvudfrågan
  frågade om) — eller när svaret vilar stegets mest avgörande punkt på
  en åsikt eller plan ("de gillade det", "vi ska"): då EN följdfråga som
  prövar just det. Aldrig för att svaret är kort; ett kort svar, en
  etikett eller ett valt förslag är alltid fullgott, och ett ärligt "vet
  inte än" på en fråga som bad grundaren ta reda på eller räkna på något
  är just det frågan skulle visa — ingen följdfråga då. Följdfrågan har
  samma format som huvudfrågan, med samma förslagsregel (jag-form, bara
  det användaren skrivit, ett ärligt "vet inte än" när något är öppet),
  men visar inget perspektiv, visas med
  "· om du vill" och har alltid
  "Gå vidare utan →", som använder det preliminära resultatet. När den
  är besvarad tvingar servern `needsFollowUp = false`. Valen i steg 5/7
  och stoppregeln har ingen följdfråga.
- Inget steg planerar längre flera frågor (§5 är ersatt). Anrop per steg:
  steg 1 = `initSkarpMap` + `readIdea` (+1 per rättelse) och vid sidan
  av `classifyIdea` (0.13, körs inte om vid en rättelse), steg 2 =
  `answerWhyYou`, steg 3 = `answerCustomer` + `findInsight`, steg 4 =
  `answerProblem`, steg 5 = `reframeProblem`, steg 6 = `buildReality` +
  `mapCompetitors`, sedan `answerReality` + `findInsight`, steg 7 =
  `proposeMechanisms`, steg 8 = `answerUnique`, steg 9 =
  `skepticAssessment` (stoppregeln kräver inget anrop), steg 10 =
  `closingSummary` + `closingAssessment` + `findInsight`; +1 vid
  följdfråga i steg 2, 3, 4, 6 och 8. Utan följdfrågor och rättelser
  blir det 18 logiska AI-anrop per session (17 före Skärpan —
  klassificeringen är det enda nya), 23 med en följdfråga i vart och ett
  av steg 2, 3, 4, 6 och 8, och 24 med dessutom en rättelse i steg 1.
  Taket är omkring 25. Ett logiskt anrop är ett `*Logic`-anrop; en
  rättande runda (0.7) är ett extra fysiskt anrop inuti det. Hittar
  researchen inget gör `mapCompetitors` inget AI-anrop alls.
- Varje anrop efter steg 1 får hela kontexten (`contextBlock`) och
  stegets fråga och svar (`qaBlock`): personprofilen, tidigare frågor
  och svar ordagrant (märkta "(valt förslag)" när svaret var ett klickat
  förslag), idékartans rättelser, insikterna med användarens reaktion
  och efter hand problemkartan, ramen, "Det här hittade vi ute i
  världen", kärnan, det unika och affären, sakerna värda att testa och
  slutkortet — med instruktionen att bygga vidare på dem och aldrig
  fråga om något som redan är besvarat. Sist i stegets systemprompt
  ligger SKÄRPA-blocket (0.13). Profilen och de visade perspektiven
  (`ctx.profile`, `ctx.lenses`) följer med i kontexten men skrivs aldrig
  ut i `contextBlock`.

### 0.4 Svara genom att klicka eller skriva

- 2–4 klickbara förslag i jag-form (högst 12 ord), byggda ENBART på det
  användaren redan skrivit eller valt, i olika vinklar och utan
  påhittade siffror, namn, årtal, titlar, erfarenheter eller fakta om
  kunden. Gäller frågan något som ännu är öppet är ett av förslagen ett
  ärligt, specifikt "vet inte än" (t.ex. "Jag har inte frågat … än").
  Visas som
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
  vad det betyder. När noteringarna är framme följer ev. ETT
  perspektivskifte om idén (`angle`, 0.13) och sist "Kort sagt:
  {ideaShort}" — det som hamnar i idékartan. Bryggan till steg 2
  (`bridge`) och steg 2:s fråga är ett par: bryggan sätter ord på vad
  idén visar OCH vad den ännu inte visar om användarens egen koppling,
  frågan ställer den skarpa frågan. Användaren kan markera en notering
  "stämmer inte" (den
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
  Speglingen och frågan under den är ett par: speglingen sätter ord på
  vad svaret visar OCH vad det ännu inte visar, frågan ställer den skarpa
  frågan och `why` säger vad svaret avgör (steg 2 → 3, 4 → 5, 7 → 8 och
  8 → 9, där steg 9 visar punkterna i stället för en fråga).
  Stoppregeln speglas av `closingSummary` (`stopRuleReflection`), som
  lead på slutkortet. Efter en insiktsskärm (före steg 4 och 7) har
  nästa skärm ingen lead — speglingen stod redan över insikten — så
  frågan där bär sin premiss själv i EN mening ("Om …, vad …?" eller
  "När …, vem …?").
- `ctx.mirror` byggs fortfarande av noteringarna (per `kind`), så steg
  4–10 får samma underlag som i fas 2.

### 0.6 Insiktsögonblick

- Tre ögonblick (`InsightMoment`): `efter-kunden` (efter steg 3),
  `efter-verkligheten` (efter steg 6) och `slutkort` (i slutkortet).
- `findInsight` (Syntetiseraren, starkare modellnivå) hittar EN konkret
  koppling, motsägelse eller ett mönster i det användaren skrivit, helst
  mellan olika steg — eller en blind fläck (`blind_flack`): något
  centralt för idén som användaren INTE nämnt, förankrat i 1–2 citat som
  visar luckan ("Du skrev 'X' och 'Y' — men ingenstans vem som …"). 1–2
  meningar (högst 45 ord) om vad det betyder för idén, med användarens
  ord inom citattecken — något en kunnig grundare INTE redan har sett,
  aldrig bara en upprepning. Tankeverktygen och de kända fallgroparna i
  SKÄRPA-blocket är linser (0.13). Aldrig allmänna råd som
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
- En blind fläck pekar ut sitt område (`gap`, dolt). Står området redan i
  klassificeringens `stated` eller i det sessionen fyllt i (kund,
  situation, problem — `derivedCoverage`) är det ingen blind fläck. Är
  insikten efter sista rundan fortfarande en sådan falsk blind fläck
  (eller en blind fläck utan `gap`), eller har den statistik, namn eller
  siffror som användaren inte gett (0.13), töms citaten — och utan citat
  visas ingen insikt.
- Efter steg 3 och 6 visas insikten som en egen skärm (`InsightCard`,
  kicker "Steg 3 av 10"/"Steg 6 av 10" + "En koppling i det du
  skrivit", "Två saker som drar isär", "Ett mönster" eller "Något du
  inte nämnt"): insikten
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
  (0.12) och går före rollbeskrivningen där de krockar: en erfaren, varm
  men rak affärsmänniska som visar det genom vad den lägger märke till
  (0.13), du-tilltal, korta meningar, inga utropstecken eller emojis,
  utgå alltid från något användaren skrivit, visa kunskap genom att
  sätta ord på det som bär eller skaver i just den här idén. Be aldrig
  om ett exempel bara för att ett svar är kort (Intervjuarens äldre
  regel) och fråga aldrig om Boggel förstått rätt (Spegelns). Hänvisa
  aldrig till egen erfarenhet eller till hur ofta något händer. Förbjudet
  bl.a.: "Det är en intressant idé", "Bra fråga", "Låt oss utforska",
  "Det finns många faktorer att tänka på", "Vilken spännande …", "Det
  låter som …", "jag har sett", "i min erfarenhet", "de flesta
  grundare", "studier visar". Inget beröm.
- `SKARP_BASE`s ton: varm, kunnig, konkret — som en erfaren
  affärsrådgivare. Boggel ersätter inte användarens rådgivare; den
  hjälper användaren att tänka klarare och komma förberedd.
- Promptexemplen gäller medvetet ANDRA idéer så att modellen kopierar
  skärpan, inte innehållet: i stegens instruktioner pensionerade
  hantverkare som lär ut reparationer, i rösten Sara med Notion och
  kalkylarket, och i tankeverktygen en återkommande ensemble
  (tandvårdskedjan, hundägarappen, barnvagnsmarknaden, cykelverkstaden
  m.fl., 0.13). En text som skriver av ett exempel stoppas av den mjuka
  kontrollen.
- Hitta aldrig på siffror, marknadsstorlekar, namn eller källor. Namn på
  alternativ och konkurrenter får BARA komma från research-underlaget
  (0.10).
- `generateSkarp` (`validators.ts`): ovanpå den hårda valideringen
  (`validateNoPlaceholders`, på svaret utan de dolda fälten) kontrolleras
  längd, en mening, frågetecken och förbjudna fraser (på allt användaren
  ser) och sedan Skärpans kontroller — ärlighet, verktygs-id, platt
  grundfråga, avskrivet exempel, blind fläck (0.13). Ett problem i
  taget; vid problem körs EN rättande runda. Efter sista rundan används
  svaret ändå och efterbearbetas (trimmas till hela meningar, citat och
  namn som inte hittas nollas) — mock-läget och en envis modell
  blockerar aldrig flödet.

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

- **Fråga:** `answerCustomer`s `nextQuestion` — vald efter
  informationsvärde inom stegets frågefält: orsaker och konsekvenser,
  och för den som redan svarat på det vem som bär kostnaden i dag, vem
  som förlorar på en lösning, när problemet inte uppstår och krafterna
  som håller kvar dagens sätt (0.13). För en annan idé: "Vem tjänar i
  dag på att småfelen blir liggande?" Ingen lead — insikten var förra
  skärmen — så frågan bär sin premiss själv.
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
  annan idé: "Om ingen kurs fanns, vad är det grannarna egentligen inte
  klarar när kranen droppar?"
- **Vem går först:** användaren (skriv själv först, 0.4).
- **Anrop:** `reframeProblem`, med eller utan `ownFrame`: 2–3
  lösningsneutrala ramar (ingen teknik, produkt eller affärsmodell) av
  typen orsak, aktör, system, abstraktion eller motsats ("Orsaken
  bakom", "Från någon annans håll", "Hela kedjan", "Ett steg upp",
  "Tvärtom"), var och en med varför den passar, talar för, tänk på (ett
  ärligt antagande), citat, spegling och kartrörelse. Verktygen i
  SKÄRPA-blocket styr vinklarna, och minst en ram ska vara en som en
  kunnig grundare inte redan har övervägt — fortfarande
  lösningsneutral. Väntan: `Läser "…"`, "Letar andra sätt att se samma
  problem …".
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
  också stegets fråga, som utgår från det som hittades, ett tankeverktyg
  och användarens egna ord (vad som skiljer idén från alternativen,
  varför dagens sätt ännu inte har ersatts, vad som talar emot). Utan
  fynd frågar den vad kunden gör i dag — men har användaren redan sagt
  det (`situation.alternativ`, eller `alternativ` i profilens `stated`)
  frågar den varför det sättet ännu inte har ersatts, och då erbjuds
  bara verktygen `att-inte-gora-nagot`, `varfor-inte-redan`,
  `varfor-nu` och `jtbd-krafter`. Vid svar: `answerReality` (+1 vid
  följdfråga) och parallellt `findInsight("efter-verkligheten")`.
  Väntan: `Läser "…"`, "Letar efter något du kanske inte sett …".
  `answerReality` skriver också steg 7:s fråga, som bär sin premiss
  själv (insikten ligger emellan).
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
  laga", "Hämta från en annan bransch", "Utgå från ett extremfall",
  "Flytta arbetet", "Bygg in det där de redan är", "Gör begränsningen
  till poängen", "Kombinera det du har") med en rubrik som är en princip
  för HUR det fungerar — aldrig en funktionslista eller en färdig
  produkt — och samma fält som i steg 5. Verktygen i SKÄRPA-blocket
  styr vinklarna, och minst en kärna ska vara en som en kunnig grundare
  inte redan har övervägt. Väntan: `Läser "…"`, "Prövar andra sätt att
  få det att fungera …". Anropet skriver också steg 8:s fråga, vald
  efter informationsvärde bland vad som gör kombinationen ovanlig och
  värdefull, vem som betalar och första vägen till kunderna, och i par
  med alternativens spegling (0.5). Den förutsätter inte vilket
  alternativ som väljs, och förslagen bygger på personprofilen, svaren
  och det som hittades — aldrig "ingen annan gör det".
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
  Speglingen sätter ord på vad svaret visar OCH vad det ännu inte visar
  — den står ovanför "Saker värda att testa". En följdfråga tar sitt
  perspektiv från ett av verktygen för `followUp`.
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
  är mest värt att testa …". 1–3 punkter, valda efter informationsvärde
  bland nio typer ("Det som måste stämma", "Det som talar emot", "Varför
  ingen redan löst det", "Lätt att kopiera", "Behövs tekniken?", "En
  känd fallgrop", "Vem betalar egentligen?", "Det som håller dem kvar",
  "Går det att leva med?"), aldrig samma typ två gånger och det mest
  osäkra och avgörande först — även det som redan sagts prövas. Varje
  punkt: en rak mening om vad som MÅSTE stämma eller talar emot, med
  användarens ord där det går (`quote`, kontrollerat i kod), och EN
  mening om det billigaste sättet att pröva just det inom några dagar —
  ett test, ingen lösning, inga påhittade siffror. Beteende och
  åtaganden (betalning, en bokad pilot, en introduktion) räknas starkare
  än åsikter och beröm, och testet frågar efter senaste gången något
  hände, inte om framtiden. "En känd fallgrop" (ur KÄNDA FALLGROPAR,
  0.13) kräver användarens ordagranna ord i `quote`; en sådan punkt utan
  verifierat citat tas bort om det finns andra punkter. Nämner en punkt
  ett alternativ ute i världen får den dess länk. Ingen positiv
  sammanfattning. `stopQuestion` ("Vad skulle få dig att byta riktning
  eller lägga ner?" anpassad till idén och till det mest avgörande i
  punkterna, med ett av verktygen för `stopQuestion`) har förslag
  byggda på punkterna, utan siffror — tröskeln lämnas till användaren.
  Stoppregeln kräver inget eget anrop.
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
  `closingAssessment` får SKÄRPA-blocket (0.13): varje områdes nästa
  steg prövar det mest osäkra och avgörande på billigaste sättet, det
  första testets bevis är beteende och åtaganden snarare än åsikter, och
  rådgivarfrågorna gäller det som är osäkert OCH avgörande, gärna en
  känd fallgrop som inte redan är besvarad. `closingSummary` har bara
  rösten.
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
  neutral, utan verktyg och perspektiv: "Vad skulle få dem att sluta med
  <alternativ>?" ("Det som håller kvar dagens sätt är det du måste
  slå.") när användaren redan sagt vad kunden gör i dag och det är
  högst 60 tecken (`situation.alternativ`), annars "Vad gör kunden i dag
  i stället?".
- Finns direkta alternativ följer EN mening om vad det betyder
  (`demandNote`, "Boggels tolkning"): att andra redan tar betalt för att
  lösa det kan betyda att det finns efterfrågan, och vad som då skiljer
  användarens idé. Därefter "OM KUNDEN INTE GÖR NÅGOT" (`doNothing`, med
  användarens ord om dagens lösning) och "DET VI INTE VET ÄN" (0–3
  luckor), båda "Boggels tolkning".
- Hittas inget körs anropet ändå: inga alternativ, en lugn rad ("Vi
  hittade inget på nätet som liknar din idé. Det kan betyda en lucka —
  eller att kunderna löser det på ett annat sätt.") och en fråga om vad
  kunden gör i dag — eller, när användaren redan sagt det, om varför det
  sättet ännu inte har ersatts (0.9, steg 6).
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
likadant överallt: den cachebara prefixen är `SKARP_BASE` + roll +
`BOGGELS_ROST` (steg 6 och 8 har ingen roll), systemprompten stegets
egen instruktion med SKÄRPA-blocket sist (0.13). Klassificeringen har
bara `SKARP_BASE` som prefix och inget block — den skriver inget
användaren ser.

| Anrop | Fil | `schemaName` / schema | Roll |
| --- | --- | --- | --- |
| `classifyIdea` | `knowledge/classify.ts` | `skarp_classify_idea` / `IdeaProfileSchema` | – (ingen röst) |
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
  `createServerFn`-wrapper; allt exporteras via `index.ts` (även
  `classifyIdea`, som inte går via `knowledge/index.ts`). Indata:
  `classifyIdea` tar `{ idea, clientId? }`; svarsstegen (2, 3, 4,
  6-svaret, 8) tar `{ context, main, followUp? }`;
  `reframeProblem`/`proposeMechanisms` tar `{ context, ownFrame? }`
  resp. `{ context, ownMechanism? }`; `buildReality`,
  `skepticAssessment`, `closingSummary` och `closingAssessment` tar
  `{ context }`; `findInsight` tar `{ context, moment }`. Skärpan har
  inte ändrat någon indata (bara nya valfria kontextfält).
- Dolda resultatfält (visas aldrig, men routen, flödestestet och evalen
  läser dem): `knowledge.offered` (vilka verktyg blocket erbjöd per
  fält), `profileUpdate` (steg 2, 4 och 6-svaret), och på visade frågor
  `assumption`, `lens` och `perspective`; på insikten `lens`,
  `perspective` och `gap`; på steg 1 `angle`.
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
  `skeptic` (`concerns` med test och citat) och `closing`. Valfria och
  aldrig utskrivna i `contextBlock`: `profile` (klassificeringen,
  uppfriskad) och `lenses` (verktygen bakom det som visats, 0.13).
- Hjälpare: `text.ts` (citatmatchning, meningsdelning och -trimning),
  `validators.ts` (`generateSkarp`, `cleanQuestion`, `settleQuestion`,
  `settleOpenQuestion`, `settleFollowUp`, `stripHidden`,
  `verifiedQuote`, förbjudna fraser och Skärpans mjuka kontroller),
  `sharp.ts` (stegens gemensamma frågeregler och `sharpChecks`),
  `knowledge/` (kunskapsbasen, urvalet och blocket, 0.13), `choice.ts`
  (`settleChoiceOptions`, `settleOwnChoice`, `userSources`), `labels.ts`
  (svenska etiketter för ramtyper, tankeoperationer, Skeptikerns typer,
  områden och nivåer — alltid i kod, aldrig av AI:n), `render.ts`
  (`renderSituationSentence`, `ORIGIN_LABELS`,
  `renderClosingCardMarkdown`), `progress.ts` (tid kvar), `map-note.ts`
  (kartrörelsen i ord), `roles.ts` (`BOGGELS_ROST`, `VOICE_EXAMPLES`).
- UI: `src/components/skarp/` (`SkarpProgress`, `StepScreen`,
  `AnswerComposer`, `IdeaReadView`, `InsightCard`, `IdeaMap`,
  `ThinkingLine`, `FocusError`, `SkarpRail`, och för steg 4–10
  `ChoiceList`, `RealityView`, `QuestionBlock`, `TestList`,
  `ClosingCard`, `FollowupDialog`), delade primitiver enligt prototypen
  i `src/components/shared/boggel-ui.tsx` (`PrimaryButton`,
  `TextButton`, `Kicker`, `NumberRomb`, `Highlight`, `GoldDash`,
  `LinedPaper`) och guldtonerna/markeringarna i `src/styles.css`.
  Visuellt facit: `design-reference/skarp/prototyp.html`.

### 0.13 Skärpan: kunskapsbas, anpassning och röst

Skärp används av grundare som redan har en idé och kan grunderna.
Skärpan ska få Boggel att se det användaren inte har sett: inbyggd
expertkunskap (en kunskapsbas av tankeverktyg), frågor valda efter
informationsvärde, en skarpare röst — och en eval som mäter det
(`docs/skarp-kvalitet.md`). Underlaget är `docs/rapport-affarsideer.md`.

Oförändrat: de tio stegen och deras ordning, skärmarna och designen
(utom perspektivetiketten och vinkeln i steg 1, båda byggda av
befintliga delar), kartan och romben, research-pipelinen, AI-lagret
(`src/lib/ai/**`), Kompass, `SKARP_BASE` och rollerna, och alla
ärlighetsregler: ordagranna citat kontrollerade i kod, "Dina ord" bara
för text användaren själv skrivit, de förbjudna fraserna (bara
utökade), "Du först" i steg 5 och 7, namnregeln i steg 6 och inga
påhittade siffror, namn eller fakta.

#### Kunskapsbasen

`src/lib/skarp/knowledge/`: `types.ts`, `tools.ts` (`KNOWLEDGE_TOOLS`),
`death-causes.ts` (`DEATH_CAUSES`), `select.ts` (urvalet), `block.ts`
(SKÄRPA-blocket), `classify.ts` (klassificeringen — den enda filen här
med serverberoenden) och `index.ts`. Resten är ren kod utan anrop.
Routen och komponenterna importerar inga verktyg — perspektivetiketten
kommer färdig i stegens resultat.

- **Tankeverktyg** (47). Vart och ett är ett perspektivskifte som en
  erfaren rådgivare gör när hen läser just den här idén — inte en
  checklista. Fält: `id` (kebab-case; modellen skriver det i `lens`),
  `name` (bara i prompten), `perspective` (etiketten användaren ser,
  vardagssvenska, högst 28 tecken, t.ex. "Vem som skriver under",
  "Dagen de byter", "Kund 1, 10 och 100"), `family` (kund och köp 7,
  problem 6, alternativ och tid 4, lösning 6, affär 6, försvar 5, risk
  och bevis 8, typspecifika 5), `ideaTypes` ("alla" eller de idétyper
  verktyget gäller), `slots` (stegen och insiktsögonblicken där det får
  användas), `addresses` (vilka osäkerheter det minskar), `tier`
  (`grund` = grundfrågan, som en erfaren grundare redan har svarat på —
  sex verktyg; `fordjupning` = nästa nivå), `signals` (ordstammar som gör
  verktyget mer relevant när de står i användarens egna ord), `when`,
  `shift` (instruktionen till modellen: vilket perspektiv, och vad
  grundaren ska behöva ta reda på, räkna på eller tänka om), ett platt
  och ett skarpt exempel (`flat`/`sharp`) och ibland en varning
  (`caution`, t.ex. "viralitet är en hypotes, inte en plan").
- **Kända fallgropar** (`DEATH_CAUSES`): 3–4 sätt som idéer av varje typ
  dör på, t.ex. pilotträsket (B2B), tappet efter första månaden (app),
  förbikoppling efter matchningen (marknadsplats), ägarens timmar är
  taket (lokal) och upphandlingen som tar längre tid än pengarna räcker
  (offentlig sektor). Var och en har en etikett, en mening om hur det
  går till och ett frågemönster på högst 16 ord.
- **Idétyper:** kundtyperna `b2b`, `b2c` och `offentlig` och formtyperna
  `saas`, `app`, `marknadsplats`, `tjanst`, `lokal`, `hardvara` och
  `fysisk_produkt`. **Osäkerheter** (16): kund, situation, problem,
  betalare, inköp, byte, alternativ, varför nu, kil, distribution, pris,
  försvar, leverans, tvåsidig, grundare och bevis — i prompterna alltid
  som svenska fraser (`UNCERTAINTY_LABELS`, t.ex. "vem som betalar och
  godkänner, jämfört med vem som använder"), aldrig som id:n.
- **Innehållsregler** (kontrolleras i `scripts/skarp-knowledge-test.ts`):
  bara heuristiker — inga andelar, frekvenser, procent eller "studier
  visar"; blocket säger "heuristiker, inte fakta". Exemplen gäller
  ANDRA idéer ur en återkommande ensemble (tandvårdskedjan där Sara är
  inköpare för tolv kliniker, appen för nya hundägare, marknadsplatsen
  för begagnade barnvagnar, cykelverkstaden, fuktmätaren för
  fastighetsägare, bokföringstjänsten för frisörsalonger,
  mattematerialet för kommunens högstadieskolor och ryggsäcken i
  återvunnen segelduk) — aldrig evalpersonernas branscher
  (`DOMAIN_STEMS`, `docs/skarp-kvalitet.md`). Ett skarpt exempel är EN
  fråga på högst 16 ord med ankaret i exempelidéns egna ord, för någon
  som redan har svarat på grundfrågan. Företagsnamn bara om användaren
  nämnt dem eller de står under "DET HÄR HITTADE VI UTE I VÄRLDEN" —
  annars "en större aktör". Verktygens ordning är urvalets tiebreak.

#### Klassificering, nivå och profil

- `classifyIdea` (`knowledge/classify.ts`, `skarp_classify_idea`,
  standardnivån) är sessionens ENDA extra logiska anrop. Den läser idén
  och gör en tyst bedömning som aldrig visas: `ideaTypes` (exakt en
  kundtyp först, sedan en eller två formtyper), `stated` (områden där
  texten ger ett konkret svar — dem frågar ingen VAD de är), `open` (2–6
  områden som är mest osäkra OCH avgörande, ordnade efter vilket svar
  som skulle ändra idén mest om det visade sig fel), `riskiestAssumption`
  (en mening om det som måste stämma — Boggels tolkning, aldrig citerad
  som användarens ord) och `level`. Bara platshållarkontrollen, ingen
  mjuk kontroll.
- **Sagt är inte belagt.** Planer, priser, kanaler och "kunderna gillar
  det" kan vara både `stated` och `open`. Bevis blir aldrig `stated` av
  åsikter eller planer, bara av beteende (betalning, återkommande
  användning, egna siffror ur verksamheten). Boggel frågar alltså inte
  om det som redan står där, men prövar hur säkert det är.
- `settleProfile` gör varje svar till en giltig profil, även mockens:
  okända värden och dubbletter bort; exakt en kundtyp först (den första
  som angavs, annars `b2c` om en formtyp är app eller fysisk produkt,
  annars `b2b`) och högst två formtyper; inget tas bort ur `open` för att
  det står i `stated`; färre än två i `open` fylls på till tre ur en
  standardordning för idéns typ (offentlig sektor: inköp, betalare,
  distribution; SaaS och B2B: betalare, inköp, distribution; app och
  B2C: byte, distribution, pris; marknadsplats: tvåsidig, kil, pris;
  tjänst, lokal, hårdvara och fysisk produkt: leverans, pris,
  distribution); antagandet kortas till en mening (högst 300 tecken);
  en ogiltig nivå blir `van`.
- **I routen** startar klassificeringen när idén skickas, utanför steg
  1:s anrop (tidsgräns 20 s; ett fel ger bara ingen profil och stoppar
  aldrig flödet). Ett pågående eller lyckat svar för samma idé
  återanvänds — klassificeringen körs inte om vid en rättelse, bara ett
  misslyckat anrop körs om när idén skickas igen. Steg 1:s anrop har
  därför ingen profil: vinkeln och steg 2:s fråga väljs bara på
  användarens egna ord. Före steg 2:s anrop väntar routen in
  klassificeringen i högst 8 s. Kommer den senare slås den ihop med det
  steg 2 redan friskat upp — steg 2:s nivå och osäkerheter går före.
- **Nivån** — `ny` (en lös tanke, kund eller situation oklar), `van`
  (kund och problem konkreta men affär, kanal eller pris oklart),
  `erfaren` (kund, situation och minst två av betalare, alternativ, pris
  och kanal beskrivna, gärna med egna observationer, kunder eller
  siffror) — bedöms om i steg 2 och verkar bara genom blocket: erfaren →
  "Ställ aldrig grundfrågan — använder du ett grundverktyg, ställ dess
  skarpa variant för någon som redan svarat på den"; van → samma utan
  "aldrig" ("Ställ inte grundfrågan i onödan"); ny → "vardagsord, en sak
  i taget — men fortfarande ett verkligt perspektivskifte". Nivån
  påverkar inte urvalets poäng.
- **Profilen friskas upp utan nya anrop** genom dolda sista fält: steg 2
  (`level`, `open`), steg 4 och steg 6-svaret (`open` — högst fyra, det
  som NU är mest osäkert och avgörande — och `riskiestAssumption`).
  Resultaten har dem som `profileUpdate`, och routen slår ihop med
  `mergeProfile`: nya osäkerheter först (inga dubbletter, högst sex),
  antagandet ersätts när det inte är tomt, nivån när den finns. Utan
  tidigare profil skapas en minimal (inga typer; nivån från steg 2,
  annars `van`). `stated` ändras aldrig efter klassificeringen, men det
  sessionen själv fyllt i räknas som sagt i urvalet (`derivedCoverage`:
  kund när målgruppen finns, situation när situationen finns, problem
  när problemkartan har minst en orsak och en konsekvens — inget annat).

#### Urvalet

`selectTools` (`knowledge/select.ts`) väljer verktygen per fält med ren,
deterministisk kod — inget AI-anrop. Samma indata ger samma verktyg i
samma ordning, så flödestest och eval kan jämföra körningar rakt av.

- **Vilka får vara med:** verktyget gäller slotten; har profilen typer
  måste verktyget gälla "alla" eller någon av dem — utan profil (steg 1,
  eller om klassificeringen misslyckats) kräver ett typbundet verktyg
  två egna signaler i användarens ord; det är inte uteslutet
  (taget av ett tidigare fält i samma block, eller av det samtidiga
  anropets fält när det gäller en insikt); och det är inte det senast
  visade perspektivet (`ctx.lenses`).
- **Poäng:** typträff +1 (+0,5 till om den gäller idéns första
  formtyp). Osäkerhet: verktygets bästa träff i profilens `open` ger
  +4/+3/+2/+1 för plats 1/2/3/senare, varje ytterligare träff +1, högst
  +6. Redan sagt (`stated` + `derivedCoverage`): ett grundverktyg vars
  alla områden redan är sagda får −2 och vinner aldrig på poäng (det
  fyller bara ut); ett fördjupningsverktyg som tar upp något som är både
  sagt och öppet får +1 ("pröva det som sagts") — utom i Skeptikern,
  slutkortet och slutkortets insikt, där det sagda prövas ändå.
  Signaler: +1,5 per distinkt ordstam som står (från ordbörjan) i
  användarens egna ord, högst +4,5. Redan visat tidigare: −5.
- **Val:** högst poäng först (lika → kunskapsbasens ordning), det bästa
  per familj så länge poängen är över noll, upp till `max` (3; 2 för
  följdfrågor, stoppfrågan och vinkeln; hårt tak 4, och en fjärde bara
  inom en poäng från den tredje). Ett redan visat verktyg kommer bara
  tillbaka om det fortfarande är bäst i slotten. Blir det färre än två
  fylls det på med det bästa som återstår, oavsett familj och poäng —
  nya före redan visade, ett avklarat grundverktyg sist.
- `deathCausesFor`: fallgroparna för formtyperna och kundtypen, varvade
  med formtypen först, högst fem. `settleLens`: modellens `lens`
  normaliseras (gemener, utan hakparentes, citattecken och inledande
  "lens:"; "", "null", "none", "ingen" och "-" betyder inget verktyg)
  och blir id + etikett bara om det är ett av fältets erbjudna verktyg,
  annars null för båda.

#### SKÄRPA-blocket och stegens frågefält

`knowledgeBlock` (`knowledge/block.ts`) bygger blocket som läggs SIST i
stegets systemprompt — aldrig i den cachebara prefixen. Underlaget är
bara det användaren själv skrivit: `userSources` plus anropets egna
svar som inte är valda förslag, egen ram eller kärna och rättelsen i
steg 1. Fälten per anrop (`KNOWLEDGE_CALLS`, `INSIGHT_KNOWLEDGE`):

| Anrop | Fält: slot (antal verktyg) | Läge |
| --- | --- | --- |
| `readIdea` | `angle`: ide (2); `nextQuestion`: varfor-du (3) | fråga |
| `answerWhyYou` | `nextQuestion`: kunden (3); `followUp`: varfor-du (2) | fråga |
| `answerCustomer` | `nextQuestion`: problemet (3); `followUp`: kunden (2) | fråga |
| `findInsight` efter-kunden | `insight`: insikt-efter-kunden (3), utan `answerCustomer`s verktyg för `nextQuestion` | insikt |
| `answerProblem` | `frameQuestion`: problemram (3); `followUp`: problemet (2) | fråga |
| `reframeProblem` | `options`: problemram (3, inget id) | alternativ |
| `buildReality` | `question`: verkligheten (3) | fråga |
| `answerReality` | `mechanismQuestion`: mekanism (3); `followUp`: verkligheten (2) | fråga |
| `findInsight` efter-verkligheten | `insight`: insikt-efter-verkligheten (3), utan `answerReality`s verktyg för `mechanismQuestion` | insikt |
| `proposeMechanisms` | `options`: mekanism (3, inget id); `nextQuestion`: unikt-affar (3) | alternativ + fråga |
| `answerUnique` | `followUp`: unikt-affar (2) | fråga |
| `skepticAssessment` | `points`: skeptikern (3, inget id); `stopQuestion`: skeptikern (2) | punkter |
| `closingAssessment` | `closing`: slutkort (3, inget id) | slutkort |
| `findInsight` slutkort | `insight`: insikt-slutkort (3), utan `closingAssessment`s verktyg | insikt |

`classifyIdea`, `closingSummary` (bara rösten), `initSkarpMap`,
`mapCompetitors` och `askSkarpFollowup` har inget block. Ett senare fält
i samma block får aldrig samma verktyg som ett tidigare.

Blocket, i ordning:

1. "SKÄRPA — gäller före exemplen ovan."
2. Profilraderna, bara när de har innehåll: "Idén (Boggels tolkning):
   B2B · SaaS. Nivå: erfaren.", "Redan sagt (fråga inte VAD — pröva hur
   säkert det är): …", "Mest osäkert och avgörande just nu: …", "Boggels
   tolkning av det som måste stämma (citera den aldrig som användarens
   ord): …" och nivåraden. Utan profil (steg 1) utelämnas de.
3. Reglerna. För varje fält med verktygs-id, "Så väljer du frågan": läs
   idén och alla svar och fråga ALDRIG om något som redan står där;
   skriv `assumption` först ('Om "<2–8 ord ordagrant ur användarens
   text>" inte stämmer, så <vad som ändras i idén>'), det mest osäkra
   och avgörande inom stegets frågefält, gärna i det senaste svaret; gör
   ett verkligt perspektivskifte med ETT av fältets verktyg, byggt på
   användarens egna ord, och skriv dess id i `lens` — säg aldrig
   verktygets eller ramverkets namn; kan grundaren svara med något hen
   redan sagt, en åsikt eller en färdig pitch är frågan för säker —
   fråga det hen måste ta reda på, räkna på eller tänka om (en fråga om
   en konkret händelse som grundaren ännu inte beskrivit räknas som
   skarp); ankaret i användarens ord står i SAMMA mening som frågan. Alternativen i steg 5/7 får inget id per alternativ, men minst
   ett ska vara något en kunnig grundare inte redan har övervägt.
   Skeptikerns punkter tar det mest osäkra och avgörande först, även det
   som redan sagts. Insikten använder verktygen och fallgroparna som
   linser och skriver id:t i `lens`, eller null. Slutkortets nästa steg,
   test och rådgivarfrågor prövar det mest osäkra och avgörande — aldrig
   ett allmänt råd.
4. Frågefältet: "Stegets frågefält: <grundfrågan> — för den som redan
   svarat på det: <det som ska prövas>." (`STEP_FIELDS`; ett per fält när
   fälten hör till olika steg).
5. Verktygen per fält. Det första fältet med verktyg får dem fullt ut
   (när, perspektivskifte, platt mot skarpt, ev. varning) med "heuristiker,
   inte fakta" i rubriken — alternativen i steg 5/7 utan frågeexemplen.
   Senare fält får dem korta (perspektivskiftet och den skarpa frågan);
   följdfrågan är märkt "bara om en följdfråga behövs; "lens" får vara
   null". Id:n skrivs utan hakparentes.
6. För Skeptikern, insikterna och slutkortet: "KÄNDA FALLGROPAR FÖR
   <TYPER> (heuristiker — påstå aldrig hur ofta något händer):" — högst
   fem för Skeptikern, tre annars.

Stegens frågefält (`STEP_FIELDS`):

| Slot | Grundfrågan | För den som redan har svarat på den |
| --- | --- | --- |
| ide | vad idén vilar på | det som avgör om den håller |
| varfor-du | erfarenhet och tillgångar | tillgång till de första tio kunderna; det grundaren sett inifrån som andra missar; det bakgrunden skymmer; vad hen har råd att förlora |
| kunden | vem och i vilken situation | användare mot köpare mot godkännare; händelsen som får dem att byta; det smalaste första segmentet; extremanvändaren |
| problemet | orsaker och konsekvenser | vem som bär kostnaden i dag; vem som förlorar på en lösning; när problemet inte uppstår; krafterna som håller kvar dagens sätt |
| problemram | hur problemet kan ramas in | undantaget, ett steg upp, tvärtom, hela kedjan |
| verkligheten | vad som redan finns | varför det inte redan ersatts; varför nu; det som talar emot; att göra ingenting |
| mekanism | kärnan i lösningen | var i vardagen; för hand först; vad som händer sedan; vad som blir kvar per order |
| unikt-affar | det unika och vem som betalar | vad som blir kvar per order; när pengarna kommer in; vad kanalen kräver; vad en större aktör inte kan kopiera |
| skeptikern | det svagaste antagandet | det som talar emot; om det gått fel om ett år; den kända fallgropen för den här typen; vad som håller kunderna kvar där de är |
| slutkort | nästa steg | det minsta testet av det mest avgörande; vad som räknas som bevis; vad grundaren har råd att förlora |

#### Frågorna i stegens prompter

Gemensamma rader i `sharp.ts`, samma i alla steg:

- "Välj efter informationsvärde inom stegets frågefält (se SKÄRPA
  nedan)" ersätter de gamla "välj det som SAKNAS"-raderna; varje stegs
  ämne står kvar.
- Varje fråga (huvudfråga, följdfråga, de öppna frågorna i steg 5/7,
  steg 6:s fråga och stoppfrågan) har två dolda fält FÖRST: `assumption`
  (antagandet frågan prövar) och `lens` (verktygets id). Formatet,
  förslagsregeln, följdfrågeregeln, paren spegling/fråga och
  premissregeln står i 0.3–0.5. Stegens platta exempel är utbytta mot
  skarpa i samma hantverkarfamilj (t.ex. steg 2: "Vilka grannar skulle
  ringa dig redan i dag om kranen droppade?"), ofta med det gamla som
  "Dåligt".
- `settleQuestion`/`settleOpenQuestion` städar frågan — en fråga kortas
  aldrig med "…" (den första frågemeningen visas hel; why högst 150
  tecken) — behåller `assumption` och gör `lens` till id + `perspective`
  med `settleLens`. Den öppna frågan i steg 5/7 får aldrig någon
  `suggestions`-nyckel.
- `stripHidden` tar bort `assumption`, `lens`, `gap`, `open`, `level`
  och `riskiestAssumption` före platshållarkontrollen, så att ett dolt
  "-" eller "okänt" inte kostar en runda.

#### Perspektivetiketter och vinkeln i steg 1

- **Etiketten** är verktygets `perspective`, satt i kod på servern och
  bara när modellens `lens` är ett av fältets erbjudna verktyg — annars
  ingen etikett. Huvudfrågor och de öppna frågorna i steg 5/7: stegets
  kicker "Steg n av 10 · <namn> · <perspektiv>". Med ett perspektiv står
  raden kvar medan Boggel arbetar (väntraden vid knappen visar redan
  det, och en kortare rad skulle få skärmen att hoppa på mobil); utan
  perspektiv ersätter "Boggel läser …" suffixet som förut. Steg 6:s fråga och
  stoppfrågan i steg 9: bara i `QuestionBlock`s kicker, "Din tur ·
  <perspektiv>"; stegets kicker är som förut. Följdfrågor visar aldrig
  något perspektiv ("· om du vill").
- **`ctx.lenses`:** när en fråga, vinkeln, stoppfrågan eller en insikt
  med ett giltigt verktyg visas sparar routen `{ step, id }` (insikter
  med steg `insikt-<ögonblick>`), aldrig samma två gånger i rad, högst
  40. Alternativens slag (ramtyp, tankeoperation) är inga verktyg.
  Urvalet väljer aldrig det senaste perspektivet och trycker ner
  tidigare.
- **Vinkeln i steg 1** (`angle`): ETT perspektivskifte om idén — ett
  påstående, ingen fråga och ingen värdering — med ett av verktygen för
  `angle`, utgående från ett ordagrant citat (2–8 ord) som står både i
  idén (eller rättelsen) och i texten; texten är EN mening, högst 30
  ord. Mjuk kontroll: en mening, inga chattiga fraser, citatet
  verifierat, ärlighetskontrollen. Efteråt visas den bara om citatet
  står ordagrant i idén eller rättelsen, texten saknar "?" och klarar
  ärlighetskontrollen — annars är `angle` null och inget visas.
  `IdeaReadView` visar den när noteringarna är framme, före "Kort
  sagt", byggd av befintliga delar: `Kicker` med etiketten (eller "Ett
  annat sätt att se det"), romben och stycket som tidssatsen använder,
  och citatet markerat med `Highlight`. Den är ingen notering: ingen
  siffra, inget "stämmer inte", inte med i `ctx.mirror`. Den avslutas
  med samma tunna linje som noteringarna, så att "Kort sagt" börjar en
  ny grupp.

#### Rösten

`BOGGELS_ROST` behåller alla tidigare regler och förbud och får:

- personan: en erfaren, varm men rak affärsmänniska som har sett
  hundratals idéer — säger det aldrig, utan visar det genom vad den
  lägger märke till; konkret, specifik och ibland obekväm när det
  behövs, men aldrig dömande; säger det användaren behöver höra, inte
  det som låter klokt; inga allmänna råd som skulle passa vilken idé som
  helst;
- hänvisa aldrig till egen erfarenhet eller till hur ofta något händer
  ("jag har sett", "i min erfarenhet", "de flesta", "oftast", "brukar",
  "vanligaste");
- hellre en obekväm fråga om det som avgör idén än en bekväm om det som
  redan är sagt; aldrig en fråga vars svar redan står i idén eller
  svaren; säg aldrig vilket ramverk eller verktyg du använder;
- avgränsningen i `SKARP_BASE` betyder att Boggel inte GER planer,
  kalkyler eller råd om marknadsföring, prissättning, logistik eller
  drift — men gärna FRÅGAR om de antaganden om pris, kanal, leverans och
  pengar som avgör om just den här idén håller;
- tre exempel platt → skarpt på en annan idé (där användaren själv
  nämnt Sara, Notion och kalkylarket), t.ex. "Vem är din målgrupp?" →
  "Vem godkänner fakturan för Notion som Sara betalar i dag — hon själv
  eller hennes chef?";
- sista regeln: alla exempel — stegens (pensionerade hantverkare),
  röstens (Sara, Notion, kalkylark) och verktygens (tandvårdskedjan,
  hundägarappen, barnvagnsmarknaden, cykelverkstaden m.fl.) — gäller
  ANDRA idéer; kopiera aldrig innehållet eller formuleringarna, bara
  skärpan. Röstens exempelmeningar finns som `VOICE_EXAMPLES` för
  kontrollen av avskrivna exempel.

#### De nya mjuka kontrollerna

I `validators.ts`. Underlaget per anrop kommer från `sharpChecks`
(`sharp.ts`): samma användartexter som blocket valde verktyg på, namnen
under "Det här hittade vi ute i världen" (i steg 6 även fyndens titlar
och utdrag innan de hamnat i kontexten), fältens verktygsetiketter och
blockets och röstens exempel. Kontrollerna är få och precisa, så att andelen rättande
rundor inte stiger märkbart; ett problem i taget, och fortfarande högst
EN rättande runda (0.7).

- **Frågeformatet** (`questionProblem`): EN mening, exakt ett "?",
  högst 16 ord och 130 tecken; `why` en mening, högst 16 ord och 130
  tecken.
- **Förbjudna fraser**, utökade med "jag har sett", "i min erfarenhet",
  "de flesta grundare", "många grundare", "studier visar", "forskning
  visar" och "statistiskt". "Jag har sett" och "i min erfarenhet" är
  bara förbjudna i Boggels egen röst — i förslagen (användarens
  jag-form) är de vanliga svar, och en insikt, spegling eller vinkel
  som citerar användaren ordagrant får återge dem.
- **Ärlighet** (`honestyProblem`): (a) statistik — "de flesta",
  "majoriteten av", "procent", "vanligaste orsaken", "oftast" eller
  "30 %"/"3 av 4", utom när det är användarens egna ord som citeras
  (båda talen står i hens text, med siffror eller bokstäver); (b) ett
  namn med versal (inte först i en mening, inte en tillåten förkortning
  som AI, IT, HR, VD, EU, CE, GDPR, LOU, B2B, B2C, SaaS, API, ROT eller
  RUT) som varken står i användarens text eller bland namnen ute i
  världen ("'X' har användaren inte nämnt — skriv 'en större aktör'
  …"); (c) en siffra som inte står i användarens text — med siffror
  eller bokstäver, "12" räknas när hen skrev "tolv" — och inte är 1, 2,
  3, 10 eller 100. En erbjuden verktygsetikett som står i sin helhet i
  texten ("dag 30") tas bort före kontrollen; etikettens enskilda ord
  och tal släpper aldrig igenom något på annat ställe. Gäller varje
  visad fråga och why (huvud- och följdfråga), vinkeln, Skeptikerns
  punkt och test, insikten, slutkortets `next` och rådgivarfrågorna.
- **Verktygs-id** (`lensProblem`): en huvudfråga (`nextQuestion`,
  `frameQuestion`, `mechanismQuestion`, `question`, `stopQuestion`)
  måste ha ett `lens` bland fältets verktyg. Följdfrågor får ha null.
- **Platt grundfråga** (`flatQuestionProblem`): frågor som "vem är din
  målgrupp", "vad gör din lösning unik", "har du konkurrenter", "vad är
  din affärsmodell", "vilket problem löser" och "hur skiljer du dig"
  flaggas bara när frågan saknar ankare — två ord i följd ur
  användarens text (minst ett av dem ett innehållsord) eller ett namn
  användaren skrivit. När användaren redan sagt vad kunden gör i dag
  räknas även "vad gör kunden i dag" och "vad gör de i dag i stället"
  som platta.
- **Avskrivet exempel** (`copiedExampleProblem`): fyra ord i följd ur
  blockets platta och skarpa exempel eller `VOICE_EXAMPLES`, med minst
  två innehållsord som inte alla står i användarens egen text (böjningar
  räknas: "månaden" när hen skrev "månad"). Gäller
  visade frågor och why, Skeptikerns punkter och insikten.
- **Blind fläck** (`blindSpotProblem`): `blind_flack` kräver `gap`, och
  området får inte stå i profilens `stated` eller i `derivedCoverage`
  (0.6). **Känd fallgrop** i steg 9 kräver användarens ordagranna ord i
  `quote` (0.9).
- Ordningen för en visad fråga: format och förbjudna fraser → ärlighet →
  verktygs-id → platt grundfråga → avskrivet exempel.

#### Anropen och budgeten

Klassificeringen är det enda nya logiska anropet: 18 per session utan
följdfrågor och rättelser (17 före Skärpan), 23 med en följdfråga i vart
och ett av steg 2, 3, 4, 6 och 8, och 24 med dessutom en rättelse i
steg 1 — taket är omkring 25 (0.3). Urvalet och blocket är vanlig kod.
En rättande runda är ett fysiskt anrop inuti samma logiska anrop.
Kunskapstestet (`scripts/skarp-knowledge-test.ts`, ren kod) prövar
innehållsreglerna, importreglerna och urvalets egenskaper; flödestestet
(`scripts/skarp-flow-test.ts`) speglar routen — klassificeringen före
steg 2, profilens uppfriskning, perspektiven, vinkeln, de erbjudna
verktygen och taket. Hur Skärpan mäts står i `docs/skarp-kvalitet.md`.

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
