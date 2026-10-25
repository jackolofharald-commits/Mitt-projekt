# Boggel – Product

## Vad är Boggel?

Ett AI-drivet vägledningsverktyg — inte en idégenerator, inte en fri chatt. Positioneringen är "styrd metod framför fri chatt": en process som tar en användare från otydlig tanke till en komplett, grundad, unik idébeskrivning.

## Målgruppen

**Primär målgrupp:** rådgivare/konsulter som hjälper andra starta eller utveckla företag (näringslivskontor, Nyföretagarcentrum, inkubatorer, fristående coacher), sålt som löpande licens.

**Sekundär målgrupp:** privatpersoner/befintliga företag, sålt per session/idé snarare än prenumeration.

## Hur produkten fungerar

**Skärp-läget** (för den som redan har en affärsidé) följer ett tio-
stegsflöde med grundprincipen "ge först, fråga sen": Boggel är en expert
som tänker med användaren och efter varje svar tillför egna perspektiv,
konkreta förslag, möjligheter och risker för idén — efter femton minuter
ska användaren ha fått många nya vinklar hen inte hade innan. Frågorna
är korta och sekundära. Användaren skriver själv först bara idén (steg 1)
och sin egen problemram och kärna (steg 5 och 7). Fullständig
specifikation, inklusive avslutningens "slutkort och test" (som ersätter
den tidigare fasta sex-punktsmallen): `docs/skarp-flode.md`.

**Kompass-läget** (för den som ännu inte har en idé) följer fortfarande
kärnflödet nedan, validerat genom två Lovable-tester:

**Riktning → Dina förutsättningar → Problem → Målgrupp → Konkurrenter & liknande lösningar → Vad gör den unik → Affärsmodell → [fast avslutning]**

Genom hela Kompass-flödet används styrda menyval (inte fri chatt), men med ett fritextfält ("eget svar") som override där användaren vill avvika. Avslutningen följer samma fasta sex-punktsmall som tidigare (idén i en mening, kort beskrivning, "så passar den dig", vad som gör den särskiljbar, svaga punkter/obevisade antaganden, konkreta nästa steg) — ändras inte av Skärp-lägets ombyggnad.

## Funktioner

- Kompass-läget: styrt frågeflöde genom de sju kärnstegen ovan, med fritextfält som override i varje steg, och samma fasta sex-punktsavslutning vid slutet av varje session.
- Skärp-läget: ett tiostegsflöde där AI:n frågar och speglar i stället för att föreslå färdiga idéer, som slutar i ett strukturerat slutkort och ett verklighetstest i stället för den gamla sex-punktsmallen. Se `docs/skarp-flode.md`.

## Framtida funktioner

Se `docs/roadmap.md` för fullständig prioritetsordning och detaljer. I korthet (extern research är redan klart, se docs/roadmap.md):

- Fri "babbel"-startpunkt innan det guidade menyflödet
- Förgreningsfråga i början av flödet: "helt ny verksamhet" vs. "ny idé inom befintligt företag"
- Egen produktfas: återkommande "innovationsstöd"-version för befintliga företag (prenumerationsmodell)
