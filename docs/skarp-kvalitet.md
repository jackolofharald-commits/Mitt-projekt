# Skärp-läget — kvalitetseval

Hur vi mäter om Skärp gör det det ska: får en grundare som redan har en
idé och kan grunderna att se det hen inte har sett. Evalen bor i repot
`gxy652fxr` (`scripts/skarp-quality-eval.ts` och `scripts/quality/`).
Flödet den kör beskrivs i `docs/skarp-flode.md` (§0, och §0.13 för
skärpan: kunskapsbasen, klassificeringen och rösten). Tankeverktygen och
testtabellen bygger på `docs/rapport-affarsideer.md`.

**Status:** evalen är byggd och prövad i `--mock` (alla fyra personerna,
`--compare`, `--pairwise`, `--regrade` och baslinjeskriptet mot `2d61206`).
**Riktiga körningar kräver en Claude-nyckel** (`SKARP_EVAL_ANTHROPIC_API_KEY`,
eller `ANTHROPIC_API_KEY`) — ingen sådan körning är
gjord än, så resultattabellen nedan är tom.

## 1. Vad som mäts

Fyra simulerade grundare går igenom hela Skärp-flödet med samma anrop, i
samma ordning och med samma kontext som `src/routes/skarp.tsx` — utom
konkurrentkartan (`mapCompetitors`, som bara ritar UI och räknas in i
anropsbudgeten utan att köras) och sparandet av slutkortet
(`finishSkarpSession`). Varje logiskt anrop har routens tidsgräns (60 s
per steg, 45 s för en insikt, 20 s för klassificeringen); ett anrop som
inte hinner eller fallerar räknas som en saknad punkt, och flödet går
vidare med en reservfråga där routen hade visat "Försök igen". En separat
bedömare sätter betyg på varje punkt Boggel visar. Huvudmåttet är
medelvärdet av **icke-uppenbar** och **specifik** över målpunkterna, med
ett bootstrap-intervall på 90 %.

Målpunkterna är allt Boggel ställer eller påstår för att skärpa idén:
frågor och följdfrågor, steg 1:s vinkel, alternativen i steg 5 och 7,
skeptikerns punkter (med test), insikterna, frågorna till rådgivaren och
det första testet. Steg 1:s noteringar bedöms också men räknas inte in —
de ska spegla, inte skärpa.

Det bedömda är alltid samma sorts text i båda versionerna: frågan med sitt
"varför", alternativets rubrik med "varför", punkten med sitt test,
insikten med sina citat. Perspektivetiketten, kickern, verktygets id
(`lens`) och förslagen ingår aldrig i det bedömda — förslagen visas för
bedömaren som sammanhang.

## 2. Kör så här

Allt körs med `npx tsx` från repots rot. Inga `npm run`-skript behövs.

| Vad | Kommando | Kostar |
| --- | --- | --- |
| Kunskapstestet (ren kod) | `npx tsx scripts/skarp-knowledge-test.ts` | – |
| Flödestestet (mock) | `npx tsx scripts/skarp-flow-test.ts --mock "<idé>"` | – |
| Evalen, gratis | `npx tsx scripts/skarp-quality-eval.ts --mock --personas saas` | – |
| Röktest | `npx tsx scripts/skarp-quality-eval.ts --smoke` | en persona, steg 1–3 och insikt 1 |
| Baslinjen | `scripts/skarp-quality-baseline.sh [2d61206] [--holdout]` | fyra (fem) personer |
| Nuvarande version | `npx tsx scripts/skarp-quality-eval.ts --label efter [--holdout]` | fyra (fem) personer |
| Bedömarbrus | `npx tsx scripts/skarp-quality-eval.ts --regrade <körning>.json` | bara bedömaren |
| Jämförelse | `npx tsx scripts/skarp-quality-eval.ts --compare <före>.json <efter>.json [--noise <brus>.json]` | – |
| Parvis jämförelse | `… --compare <före>.json <efter>.json --pairwise` | bara bedömaren |

Flödestestet utan `--mock` väljer leverantör precis som appen: med
`ANTHROPIC_API_KEY` i skalet blir det en hel, riktig och betald
Claude-session (plus Perplexity och Supabase om de variablerna finns) som
aldrig hamnar i huvudboken. `SKARP_EVAL_ANTHROPIC_API_KEY` läser det inte. Som kontroll körs det därför alltid med `--mock`: flaggan tar
bort `ANTHROPIC_API_KEY`, `AI_API_KEY`, `AI_BASE_URL`, `PERPLEXITY_*` och
`SUPABASE_*` och sätter `AI_PROVIDER=mock` innan något under `src/` laddas,
och räknas inte som en del av idétexten.

Flaggor:

- `--personas saas,app,marknadsplats,lokal` — urvalet (standard alla fyra).
  `--holdout` lägger till hållen-utanför-personan (rapporteras separat).
- `--concurrency 2` — personer som körs samtidigt.
- `--persona-model claude-haiku-5-5`, `--grader-model claude-opus-5-5`.
- `--research off|live|replay` — standard `off` (research avstängd).
  `live` kräver `PERPLEXITY_API_KEY` och spelar in fynden per persona;
  `replay` lämnar tillbaka personans inspelade fynd utan nätverksanrop.
- `--record` / `--no-record` — spela in alla AI-svar i
  `eval-results/cache/` (standard på för riktiga körningar).
  `--replay` spelar upp ett inspelat svar för en identisk förfrågan och går
  live vid en miss (standard i förbättringsrundor, se §8).
- `--max-usd N` (standard prognosen × 1,3), `--budget-total 15`. Taket för
  en körning är aldrig högre än det som återstår av `--budget-total` (§7).
- `--out eval-results`, `--cache-dir <katalog>`, `--verbose` (visar
  AI-lagrets loggrader i terminalen; de hamnar alltid i körningens `.log`).
- `--persona-sees-suggestions` — variant där personan ser förslagen och får
  välja ett ordagrant (skickas då som `source: "forslag"`).
- `--mock` gör också `--compare … --pairwise` och `--regrade` gratis, även
  för riktiga resultatfiler — svaren märks då MOCK och bokförs inte.

Nyckeln läses helst från `SKARP_EVAL_ANTHROPIC_API_KEY`, som går före
`ANTHROPIC_API_KEY` och bara läses av evalen. Med ett eget namn plockas
nyckeln inte upp av något annat i en molnsession som letar efter
`ANTHROPIC_API_KEY`. Evalen kopierar den till `ANTHROPIC_API_KEY` i sin egen
process innan något under `src/` laddas. I en molnsession läggs den in i
miljöns inställningar (under Network secrets där det finns, annars som
miljövariabel) och gäller från nästa session — klistra aldrig in den i
chatten.

Utan nyckel och utan `--mock` skriver skriptet vad som behövs
och avslutas med kod 2 (likaså `--research live` utan
`PERPLEXITY_API_KEY`). En riktig körning (även `--regrade` och
`--pairwise`) börjar med en gratis nyckelkontroll (`GET /v1/models`, inga
tokens): godtas inte nyckeln (401/403), eller går Claude inte att nå,
avslutas skriptet med kod 2 innan något körs eller bokförs.
Budgetspärren och kravet på `--smoke` före den första hela körningen
avslutar med kod 3, en vägrad jämförelse med kod 1.
Ctrl-C, SIGTERM eller SIGHUP (terminalen stängs) under en riktig körning
(även `--regrade` och `--pairwise`) bokför det som hunnit kosta och
avslutar med kod 130 (§7).

Under en eval stängs Supabase av (alla `SUPABASE_*` tas bort ur miljön),
och prisöverskrivningen `ANTHROPIC_PRICE_*_PER_MTOK` tas bort: den gäller
alla modeller lika och skulle felprissätta bedömaren och personsimulatorn,
så huvudboken prissätter alltid med priset per modell.
OpenRouter-reserven stängs också av: `AI_API_KEY` och `AI_BASE_URL` tas
bort, så att AI-lagrets kedja slutar vid Claude och Claude-reserven — ett
svar från en annan leverantör loggas inte som `[ai-cost]`, prissätts inte
och får aldrig bedömas som Boggels. En riktig körnings rubrik säger det. I
`--mock` tas även `ANTHROPIC_API_KEY`, `SKARP_EVAL_ANTHROPIC_API_KEY` och
`PERPLEXITY_*` bort och
`AI_PROVIDER=mock` sätts. Allt detta görs innan något under `src/`
importeras. I `--mock` svarar mock-leverantören med platshållare, personan
svarar med meningar ur sina egna fakta och bedömaren ger alla punkter 3:
**siffrorna betyder ingenting**, men hela kedjan körs.

Varje körning skriver `eval-results/skarp-quality-<label>-<ÅÅÅÅMMDD-TTMM>`
med `.json` (allt), `.md` (sammanfattningen), `.log` (alla loggrader,
märkta med persona) och `.diff` (de lokala ändringarna). `eval-results/`
är git-ignorerad.

### Baslinjen

`scripts/skarp-quality-baseline.sh` lägger upp `2d61206` (före
kunskapsbasen) i en tillfällig `git worktree`, kopierar in evalens
nuvarande filer och länkar `node_modules`. Först körs `--mock --personas
saas` där — misslyckas den körs inget mer — och sedan den riktiga
körningen med `--label fore`, där resultat, huvudbok och inspelningar
hamnar i huvudutcheckningens `eval-results/`. Worktreen tas alltid bort.
Extra argument skickas vidare (`--holdout`, eller `--mock` för en
torrkörning utan nyckel).

Evalens kod når allt som är nytt sedan `2d61206` (klassificeringen,
kunskapsbasen, `mergeProfile`, `profileUpdate`, `lens`, `perspective`,
`angle`, `knowledge.offered`) genom `scripts/quality/compat.ts`, med
dynamiska importer och försiktig läsning. Mot baslinjen blir körningen
därför giltig utan klassificering, perspektiv och vinkel.

## 3. Personerna

Fyra grundare som redan har en idé och kan grunderna
(`scripts/quality/personas.ts`):

| Id | Grundare | Idé | Förväntad typ |
| --- | --- | --- | --- |
| `saas` | Lina, f.d. lönekonsult | tidrapport och löneunderlag för byggföretag med 20–150 anställda | B2B · SaaS |
| `app` | Omar, UX-designer, tvåbarnsförälder | app för föräldrar med barn 1–4 år: veckans middagar av det som finns hemma, stegvis tillvänjning | B2C · app |
| `marknadsplats` | Elin, f.d. inköpare på en restauranggrupp | restauranger i Göteborg köper direkt från gårdar inom tio mil, samlad leverans | B2B · marknadsplats |
| `lokal` | Johan, elektriker med egen firma | laddboxar för bostadsrättsföreningar i Uppsala, fast pris per plats och serviceavtal | B2B · lokal |
| `fysisk` (utanför träningen) | Maja, produktdesigner | modulärt förvaringssystem för små lägenheter, direkt till konsument | B2C · fysisk produkt |

Alla förväntas klassas som `erfaren`. Varje persona har tre lager:

- **Idén** (steg 1-texten, 90–170 ord): grunderna står redan där — kund,
  situation, problem, alternativ, pris eller kanal och lite egna belägg.
- **Det grundaren vet men inte skrev** (8–12 saker): sådant en kunnig
  grundare redan har övervägt, med en hållning — även läroboksfrågorna för
  just den sortens idé. Avslöjas bara när en fråga ber om det, högst en sak
  per svar. Bedömaren ser listan, och enligt dess regler (§5) får en punkt
  som bara lockar fram något av det högst 3 på icke-uppenbar.
- **Blinda fläckar** (3–5): verkliga andra ordningens saker grundaren inte
  har tänkt på. Det är dem en skarp tankepartner ska hitta. Simulatorn
  känner igen när en fråga träffar en, och rapporten visar andelen per steg.

`DOMAIN_STEMS` i samma fil är personernas branschord. Kunskapstestet
kontrollerar att de aldrig förekommer i kunskapsbasen, rösten eller
stegens prompter — en förbättring som bara fungerar för personernas egna
branscher är överanpassning.

## 4. Simulatorn

En billig modell (`claude-haiku-5-5`) spelar grundaren. Den ser personan,
samtalet hittills (sina egna svar och Boggels frågor) och den aktuella
frågan med "varför" — aldrig förslagen (utom med
`--persona-sees-suggestions`). Den svarar med 1–4 meningar, högst 400
tecken: det den vet, en hållning, eller ärligt "Det vet jag inte än." /
"Det har jag inte räknat på." Den kommenterar aldrig frågan ("bra fråga")
och hittar inte på nya fakta. Träffar en fråga en blind fläck säger den
att det inte är genomtänkt och ger sin magkänsla.

I steg 5 och 7 skriver personan sin egen ram/kärna först och väljer sedan
att behålla den eller ta ett av Boggels alternativ. Insikterna efter steg
3 och 6 får "Stämmer" eller "Inte riktigt" med en rättelse, precis som i
appen. Det dolda (vilka fakta som avslöjades, vilken blind fläck som
träffades) når aldrig Boggel eller bedömaren.

Misslyckas ett simulatoranrop två gånger (fel modell-id, 429, timeout) blir
det ett reservsvar: "Det vet jag inte än.", den egna formuleringen i steg 5
och 7, eller "Stämmer" på en insikt — och ett sådant går inte att skilja
från ett riktigt. Därför räknas misslyckade svar, val och reaktioner per
persona. Antalet står på personans rad och i sammanfattningen, rapporten
har en egen rad för det, och när simulatorn ger upp syns det direkt i
terminalen. **Två eller fler i en persona gör körningen "ofullständig –
jämförs inte"**, och `--compare` vägrar då jämföra den.

## 5. Bedömningen

En separat, starkare modell (`claude-opus-5-5`) bedömer. Exakt två anrop
per persona med fasta gränser: **A** = steg 1–5 och insikten efter
kunden, **B** = steg 6–10 och de två senare insikterna. Båda läser EN
kronologisk utskrift: idén, sedan steg för steg det som visades men inte
bedöms (`[VISADES – BEDÖMS INTE]`: speglingar, research-alternativ,
"vad det betyder", luckor, förslag), varje bedömd punkt med sitt id på
sin plats och grundarens svar efter den, plus "Saker grundaren redan
övervägt (men inte skrivit)".

Regler i prompten:

- Bedöm varje punkt bara mot det som står FÖRE den — det grundaren svarar
  efteråt får inte påverka betyget.
- Längd är ingen kvalitet.
- icke-uppenbar ≤ 3 om punkten i sak motsvarar något grundaren redan
  övervägt, eller något en kompetent allmän chattbot skulle fråga oombedd
  utifrån samma material.
- Innan betygen skriver bedömaren `chatbotBaseline`: två frågor per steg
  som en kompetent allmän chattbot skulle ställa. Det är ribban.

Per punkt, i den här ordningen: `motivering` (alltid, högst 20 ord),
`redanBesvarad`, `perspektivskifte`, sedan fyra heltal 1–5:

| Kriterium | 1 | 3 | 5 |
| --- | --- | --- | --- |
| specifik | kunde ställas till vilken idé som helst | rätt bransch, men allmän | bygger på en konkret detalj grundaren själv skrev |
| icke-uppenbar | står redan i idén / det varje chattbot frågar | rimlig fördjupning som grundaren troligen redan tänkt på | en andra ordningens följd grundaren inte sett |
| användbar | inget att göra med det | något att fundera på | leder direkt till ett test, ett räknestycke eller ett beslut |
| ej generisk | mallfras, floskel | delvis konkret | helt konkret, i grundarens egna ord |

Ankarna i prompten är skrivna på en idé som inte används någon annanstans
(en tjänst som sköter kontorsväxter åt små företag). Efteråt gäller:
`redanBesvarad` → icke-uppenbar högst 2. Bedömaren får ett försök till på
evalnivå; punkter utan betyg märks och räknas inte in. Prompten ber om
id:t utan hakparenteser, och evalen jämför ändå id:n utan hakparenteser,
blanksteg och skiftläge — ett eko som `[SAAS-07]` räknas inte som ett
saknat betyg (som annars hade kostat ett nytt bedömaranrop). Prompten
innehåller aldrig etiketter, commit, version eller "före/efter", och dess
hash står i resultatfilen — `--compare` vägrar jämföra körningar med
olika hash.

## 6. Rapporten

- **Huvudmått:** icke-uppenbar och specifik över målpunkterna, med 90 %
  bootstrap (personer dras om, sedan punkter inom varje person; fast frö).
- **Per steg** (1–10, insikt-3, insikt-6, insikt-slutkort): de fyra
  kriterierna, n, intervall för de två nyckelkriterierna, medellängd,
  perspektivtäckning (andel personer med minst en punkt med giltig `lens`
  och `perspektivskifte`) och andel redan besvarade. **JA** om medel ≥ 4,0
  och undre gräns ≥ 3,5; **OSÄKERT** om intervallet innehåller 4,0;
  annars **NEJ** — för båda nyckelkriterierna, och det sämsta gäller.
- Per persona; per verktyg (`lens`, n ≥ 2, sämst först); per anrop
  (`generatedBy` = schemanamnet); per punkttyp; andel träffade blinda
  fläckar per steg; förväntat/visat/saknat per steg; insikter som visades;
  klassificeringen mot förväntan; homogenisering (samma verktyg i ett steg
  hos minst tre av fyra personer, eller frågor med ordlikhet ≥ 0,5); de
  fem svagaste punkterna (bara som illustration); tid (p50/p95) och utfall
  per anrop; logiska anrop per session; ärlighetskontroller (insiktscitat
  som står ordagrant i användarens text, vinklar med verifierat citat).
- Resultatfilen har också ursprunget: commit, om trädet har lokala
  ändringar (sha256 av `git diff HEAD`, diffen sparad bredvid),
  `ANTHROPIC_MODEL`, `AI_MODEL_STRONG`, `ANTHROPIC_FALLBACK_MODEL`,
  `ANTHROPIC_TIMEOUT_MS`, person- och bedömarmodell, bedömarens prompt-hash
  och research-läge — och vilken modell som faktiskt svarade på varje
  Boggel-anrop (svar från reservmodellen flaggas). En tom variabel, som
  `AI_MODEL_STRONG=` i `.env.example`, räknas som osatt precis som i
  AI-lagret, så den starka nivån flaggas inte i onödan.

`--compare` ger en tabell per steg för de fyra kriterierna, före → efter,
och hur många personer som blev bättre i steget, bara på punkttyper som
finns i båda körningarna (t.ex. räknas vinkeln inte mot baslinjen). Den
vägrar om bedömarens hash, person- eller bedömarmodellen eller research-
läget skiljer sig, om den ena körningen är mock och den andra inte, eller
om en körning är "ofullständig" (en persona hoppades över eller avbröts,
eller simulatorn misslyckades två gånger eller fler i en persona) eller en
`--smoke`.
`--pairwise` gör ett bedömaranrop per persona med alla stegpar; A och B
lottas per par (fast frö, placeringen sparas) och frågan är "Vilken är
skarpast för just den här grundaren: A, B eller lika?".

## 7. Kostnad och budget

Varje persona körs i en egen `AsyncLocalStorage`, och AI-lagrets egna
loggrader (`[ai-cost]`, `[research] … kostnad`, `[ai] ✗`, reservraderna
och `[skarp] … mjuk kontroll runda`) räknas per persona och schemanamn i
fyra kategorier: Boggel, personsimulatorn (`eval_persona_*`), bedömaren
(`eval_grade*`) och research. Okända priser räknas med det högsta kända
priset som tak och redovisas som "minst X, högst Y". Logiska Boggel-anrop
(ett per `*Logic`-anrop) räknas av evalen själv och jämförs med taket ~25
(`mapCompetitors` körs inte i evalen men räknas in). Uppmätt i `--mock`
utan följdfrågor: 17 logiska anrop mot `2d61206`, **18 i nuvarande
version** (klassificeringen är det enda nya anropet). En följdfråga ger
ett svarsanrop till, och i steg 3 och 6 startar routen — och evalen, som
speglar den — dessutom om insikten med följdsvaret; det ersatta
insiktsanropet väntas aldrig in men är gjort och räknas. Med en följdfråga
i vart och ett av de fem stegen som kan ha en blir det därför **25 logiska
anrop** i nuvarande version (fem svarsanrop och två omstartade insikter
till; 24 mot `2d61206`), `mapCompetitors` inräknad, och **26 med dessutom
en rättelse i steg 1** — alltså precis vid taket. Rapportens rad "Logiska
anrop per session" räknar på samma sätt och visar per persona hur många
följdfrågor och omstartade insikter som ingår (⚠ först över 25).
Flödestestet visar 17 i läget utan följdfrågor: utan research-fynd gör
`mapCompetitors` inget AI-anrop, och flödestestet räknar bara anrop som
faktiskt görs.

Varje riktig körning skrivs i `eval-results/ledger.jsonl` — även
`--regrade` och `--pairwise`, som kontrollerar budgeten på samma sätt med
en prognos för bara bedömaren. Rader läggs bara till, filen skrivs aldrig
om: under körningen en preliminär rad efter varje klar persona och
löpande medan pengarna går åt (högst var tionde sekund), sedan den
slutliga. Vid läsning gäller den senaste raden per körnings-id, så samma
pengar räknas aldrig två gånger, och två körningar mot samma huvudbok kan
inte skriva över varandras rader. Avbryts körningen — Ctrl-C, SIGTERM,
SIGHUP, ett fel — skrivs det som hunnit kosta med etiketten
`<etikett> avbruten`, och vid en signal avslutas processen med kod 130.
Ett hårt avbrott som inte går att fånga (`kill -9`, strömavbrott) tappar
högst de senaste ungefär tio sekundernas förbrukning. Innan den slutliga
raden skrivs väntar körningen (högst två minuter) in AI-anrop som
fortfarande pågår — ett anrop som runnerns tidsgräns slutat vänta på, eller
en ersatt insikt, kostar ändå när det blir klart. Varje rad har också
trädets version (de nya delarna sedan `2d61206`, samma lista som i
rapportens rubrik) och commit, om körningen blev komplett, antalet anrop
per schemanamn och de logiska anropen per schemanamn. Preliminära och
avbrutna rader räknas i totalen men är aldrig underlag för en prognos.

Före en körning räknas en prognos fram: den senaste **kompletta** hela
körningen **av samma version** (samma nya delar i trädet — en
förbättringsrunda som bara ändrar antalet verktyg räknas som samma
version; baslinjen `2d61206` med sina ungefär 1,7 × mindre prompter
budgeterar alltså aldrig den nuvarande versionen), helst en utan
uppspelade svar, × antal × 1,2. Finns ingen, används senaste uppmätta
`--smoke` (helst av samma version) uppräknad till en hel session × 1,2:
kostnaden per logiskt Boggel-anrop med mjuka rundor och schemaförsök
inräknade, steg 4–10 (som röktestet inte når) som 1,5 × medlet av de
uppmätta, och bedömningen som hela del A plus del B ≈ 3,5 × röktestets
del A. Ett `--smoke` av en annan version ger aldrig mindre än 1,2 USD per
persona. Annars försiktigt 1,2 USD per persona. En ofullständig körning
(en persona hoppades över eller föll bort) är aldrig underlaget — men är
den nyare än underlaget kan det den hann mäta per persona (× 1,2) bara
höja prognosen, aldrig sänka den. Körningen startar inte om huvudboken
plus prognosen överstiger `--budget-total` (standard 15 USD).

Före varje persona — i en körning, i `--regrade` och i `--pairwise` —
måste det som redan kostat (även det pågående personer hunnit använda),
det pågående personer har **kvar** av uppskattningen och uppskattningen
för den nya rymmas under både `--max-usd` (standard prognosen × 1,3) och
det som återstår av `--budget-total`. Huvudboken läses om varje gång, så
att en annan körning mot samma huvudbok räknas in. Uppskattningen är
prognosen tills en persona är klar, sedan det uppmätta medlet. En persona
som inte hinner bli klar startas aldrig; en körning som saknar en persona
märks "ofullständig – jämförs inte", och en `--regrade` eller `--pairwise`
som hoppade över någon säger det i sin sammanfattning. En `--regrade` av
ett `--smoke` (bara del A för steg 1–3) är aldrig underlag för prognosen
för en hel omräkning.

`--smoke` spärras mot sin egen uppskattning (ungefär en halv session,
taket blir den × 1,3) och krävs före den första hela körningen. Det räknas
bara som mätning om personan faktiskt kördes och minst ett Boggel-anrop
gick fram — ett röktest som hoppade över sin persona eller där alla anrop
misslyckades låser inte upp något, och en körning där varje persona
hoppades över skriver ingen rad alls. Röktestet skriver ut uppmätt
kostnad per anropstyp och per logiskt anrop och en prognos för en hel
körning.

`--record` sparar varje AI-svar under en nyckel (sha256 av modell, den
starka modellen, modellnivå, schemanamn, systemprompt, cachebar prefix,
meddelanden och schemats form). `--replay` lämnar tillbaka det sparade
svaret för en identisk förfrågan (loggas `[eval-replay]`, $0, räknas för
sig) och går live vid en miss. `--regrade` spelar aldrig upp — det är en
ny bedömning.

## 8. Iterationsprotokollet

Körs när en nyckel finns. Före varje runda: kunskapstestet och
flödestestet med `--mock` gröna (utan flaggan blir flödestestet en betald
Claude-session utanför huvudboken, se §2).

1. `npx tsx scripts/skarp-knowledge-test.ts` och
   `npx tsx scripts/skarp-flow-test.ts --mock "<idé>"` gröna.
2. `--smoke`.
3. Baslinjen: `scripts/skarp-quality-baseline.sh` (`fore`, med
   `--holdout` om budgeten räcker).
4. Nuvarande version: `--label efter`.
5. `--regrade` baslinjen en gång — bedömarbruset.
6. Förbättra utifrån sammanställda mönster (per verktyg, per anrop, per
   punkttyp), aldrig utifrån en persons innehåll — en förbättring ska vara
   en allmän heuristik. Kör om med `--replay`. Behåll en ändring bara om
   totalskillnaden är ≥ max(0,25, 2 × bedömarbrusets SD)
   (`--compare … --noise <regrade>.json` räknar ut det).
7. Sluta när icke-uppenbar och specifik båda är ≥ 4,0 utan något steg på
   NEJ, efter tre förbättringsrundor, eller när huvudboken inte rymmer en
   körning till (budgeten går före antalet rundor).
8. Slutkörning med `--holdout`; för in före/efter (absolut och parvis) och
   kostnaden per Boggel-session nedan.

## 9. Resultat

Fylls i när riktiga körningar finns.

| Körning | Datum | Commit | icke-uppenbar (90 % KI) | specifik (90 % KI) | Steg på NEJ | Kostnad | Kommentar |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `fore` (2d61206) | – | – | – | – | – | – | inte körd (kräver en Claude-nyckel, §2) |
| `efter` | – | – | – | – | – | – | inte körd |
| Utanför träningen, före/efter | – | – | – | – | – | – | inte körd |

Parvis jämförelse (vinst/oavgjort/förlust per steg): inte körd.
Bedömarbrus (SD): inte uppmätt.

### Kostnad per Boggel-session

Logiska anrop räknas som i §7: `mapCompetitors` och omstartade insikter
inräknade.

| Version | Logiska anrop | Kostnad per session | Varav mjuka rundor |
| --- | --- | --- | --- |
| `2d61206` | 17 (mock, utan följdfrågor); 24 med en följdfråga i alla fem stegen | – | – |
| nuvarande | 18 (mock, utan följdfrågor); 25 med en följdfråga i alla fem stegen, 26 med en rättelse i steg 1 | – | – |

## 10. Det som saknas

- **Riktiga körningar kräver en Claude-nyckel** (§2). Ingen sådan nyckel
  fanns där evalen byggdes, så röktestet, baslinjen, den nuvarande
  versionen och bedömarbruset är inte körda. Allt annat är prövat i
  `--mock`.
- `--research live` kräver `PERPLEXITY_API_KEY` (+ endpoint och modell).
  Utan den körs evalen med research avstängd, och steg 6 bygger då bara
  på det användaren skrivit — samma för före och efter.
- Prisuppgiften för reservmodellen (`claude-haiku-4-5-20251001`) saknas i
  `src/lib/ai/log.server.ts`; svar från den redovisas som "minst X, högst Y".
- Bara anrop med en `[ai-cost]`-rad prissätts. Ett försök som underkänns
  av schemat eller avbryts av tidsgränsen kan vara debiterat men räknas
  som $0 (det syns bara som ✗ i tabellen per anrop), så huvudboken kan
  ligga något under det Anthropic faktiskt debiterar — lämna marginal mot
  `--budget-total`.
