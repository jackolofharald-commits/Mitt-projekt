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

Skärp-läget är INTE ett styrt menyflöde längre (det var det gamla
flödet). Det är en tankemotor: AI:n frågar, speglar, strukturerar och
utmanar — den skriver aldrig färdiga idéer eller säljtext åt användaren,
och generell AI-input visas aldrig innan användaren själv skrivit något
i de steg där det spelar roll ("Du först", med en flyktväg för den som
kör fast). Flödet har tio steg (Din idé → Varför du → Kunden i
situationen → Utvidga problemet → Välj problemram → Verkligheten →
Lösningskärnan → Det unika och affären → Skeptikern → Slutkort och
test), bygger en personprofil i steg 2 som följer med genom resten av
sessionen, och märker varje påstående i slutkortet med sitt ursprung
(användarens egna ord, AI:ns tolkning, eller extern källa med länk).
Sedan fas 3 (alla tio steg ombyggda) är Skärp en arbetsyta med ett steg
i fokus i stället för en chattlogg: en huvudfråga per steg och högst en
följdfråga, det användaren sagt växer fram i en idékarta med nio fält i
railen (klickbara för att rätta), och Boggel lyfter insiktsögonblick —
en konkret koppling, motsägelse eller ett mönster i användarens egna
ord (efter steg 6 även i det som hittats ute i världen) — efter steg 3,
efter steg 6 och i slutkortet. Slutkortet har inga sifferbetyg: det
börjar med det som är starkt, ger varje område "Starkt", "På god väg"
eller "Behöver mer underlag" med en mening om nästa steg, och samlar
"Saker värda att testa", ett första test och frågor att ta med till
rådgivaren. Sedan Skärpan (§0.13) tänker Boggel med en inbyggd
kunskapsbas — tankeverktyg och kända fallgropar per idétyp, bara
heuristiker och aldrig statistik — och väljer frågor efter
informationsvärde för grundare som redan kan grunderna. Ett tyst
klassificeringsanrop efter steg 1 (sessionens enda extra anrop: 18
logiska anrop utan följdfrågor, tak ~25) bedömer idétyp, vad som redan
är sagt (sagt är inte belagt), vad som är mest osäkert och avgörande och
hur van användaren är, och vanlig kod väljer sedan deterministiskt
vilka verktyg varje steg får i SKÄRPA-blocket. Varje huvudfråga är EN
mening på högst 16 ord med ankaret i användarens egna ord och visar sitt
perspektiv som en diskret etikett, nya mjuka kontroller stoppar
statistik, påhittade namn och siffror, platta grundfrågor och
avskrivna exempel, och kvalitetsevalen i `docs/skarp-kvalitet.md`
mäter om det blir skarpare. Se §0 i `docs/skarp-flode.md`, som gäller
före resten av dokumentet.
Fullständig specifikation: `docs/skarp-flode.md`.

Kompass-läget är oförändrat och beskrivs inte här.
