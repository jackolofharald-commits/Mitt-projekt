# Designöverlämning: Kompass-läget och Skärp-läget

Den här mappen innehåller de två godkända prototyperna från Claude Design. De är **source of truth för design och upplevelse** i Boggel: layout, typografi, färger, animationer, övergångar och hur interaktionerna känns.

```
design-reference/
  kompass/prototyp.html   ← Kompass-läget, koncept 2a "Kompassen lyssnar"
  skarp/prototyp.html     ← Skärp-läget, koncept 2b "Kartan"
  HANDOFF.md              ← den här filen
```

## Så kör du prototyperna

Prototyperna är körbar kod (React via en liten runtime i `support.js`). De måste serveras över HTTP, inte öppnas som fil, och behöver internet för att ladda React från unpkg.

```bash
cd design-reference
npx serve .          # eller: python3 -m http.server 8000
```

Öppna sedan `kompass/prototyp.html` och `skarp/prototyp.html` i webbläsaren. Tryck på **"Skriv ett exempel åt mig"** för att se Boggel reagera medan texten skrivs, och klicka dig igenom alla steg fram till sammanställningen.

Konsolfel av typen `<path> attribute d: Expected moveto ... "{{ aTicks }}"` är ofarliga. De uppstår innan mallen renderas.

Prototypfilerna är inte appkod och ska inte importeras eller byggas vidare på direkt. Läs dem som specifikation.

## Koncepten

**Kompass-läget, 2a "Kompassen lyssnar".** Nålen är Boggels uppmärksamhet. Den rycker till när användaren skriver något viktigt och sveper till varje mening Boggel stryker under. Varje fynd fäster som en punkt på kompassen. När användaren väljer en riktning vrids kompassen så att riktningen blir norr, och den gula sektorn smalnar för varje steg.

**Skärp-läget, 2b "Kartan".** Det användaren skriver flyttar dem. Varje understruken mening blir en punkt på kartan och drar positionen åt sitt håll, så användaren ser exakt hur svaren påverkar riktningen. Vägarna framåt utgår från där de står. Kartan zoomar in för varje steg och zoomar ut i sammanställningen, där hela spåret syns.

**Gemensamt designspråk (gäller båda lägena):**
- DM Serif Display är Boggels röst, Fira Sans är användarens, Fira Mono för etiketter och steg.
- Gult (`#F4C530`) markerar bara det som är valt eller viktigt.
- Streckad understrykning = Boggel har noterat något. Gul understrykning = Boggel har läst och förstått det.
- Romben är en punkt i resan. Nålen (Kompass) eller positionen (Kartan) visar riktningen.
- Inga chattbubblor och inga färgkodade kort. Boggel talar genom rubriker, marginalanteckningar och korta repliker direkt i flödet.

## Viktigt att känna till innan du bygger

1. **Skärp-prototypen visar Kompass-lägets innehåll.** Alla koncept i Design-rundan togs fram med Kompass-flödet (fritext först). För Skärp-läget ska du använda 2b:s visuella koncept, kartan och interaktionerna, men anpassat till Skärp-lägets flöde, där användaren redan har en idé. Fundera på vad kartans axlar och punkter representerar när utgångspunkten är en befintlig idé, och föreslå det för oss innan du bygger.
2. **Stegen i prototyperna är förkortade.** Prototyperna visar Dina tankar → Riktning → Problemet → Förutsättningar → Konkurrenter → Sammanställning. Det riktiga flödet och den fasta avslutningsmallen står i CLAUDE.md och gäller. Designen ska bära det riktiga flödet, inte tvärtom.
3. **Innehållet är påhittat.** Exempeltexter, fynd, riktningar och sammanställning är hårdkodade i prototyperna. I appen kommer de från kärn-AI:t och research-lagret enligt CLAUDE.md.
4. **Tidigare designkällor är ersatta.** Där CLAUDE.md säger att Lovable-skissens design ska bevaras gäller nu dessa prototyper för design och upplevelse.

## Arbetsordning

1. Kör och läs igenom båda prototyperna. **Sammanfatta för oss hur varje flöde fungerar steg för steg, och hur du tänker anpassa 2b till Skärp-läget, innan du bygger något.**
2. Identifiera det som är gemensamt för lägena (typografi, färgtokens, understrykningar, romber, stegindikator, rörelser) och bygg det som ett delat designsystem, inte två separata kopior.
3. Återskapa upplevelsen troget i den riktiga applikationen. Förbättra inte designen på eget initiativ. Om något inte går att bygga som det är, fråga först.
4. Koppla flödena till backend enligt CLAUDE.md. Fasta regler gäller fortfarande, bland annat avslutningsmallen och att aldrig hitta på siffror eller namn.
