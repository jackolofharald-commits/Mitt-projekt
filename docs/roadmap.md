# Boggel – Roadmap

## Funktionsbacklogg, i prioritetsordning

1. **Klart:** Extern research (Perplexity Agent API) är inkopplad i kärnflödet, med cachning, bakgrundsstart och skydd mot dubbla/samtidiga anrop. I Skärp-läget körs två sökningar i bakgrunden (se `docs/skarp-flode.md` §0.17 och §0.18): en bred när idén skickas och en riktad när första kundgruppen är vald i steg 4. De kondenseras till en marknadsbild som följer med i alla senare steg, och steg 6 ("Verkligheten") och konkurrentkartan läser fynden ur cachen. Ingen separat "problem"-sökning körs i Skärp-läget. I Kompass-läget (oförändrat flöde) fortfarande i problem- och konkurrentstegen.

   Bevisat nödvändigt: i ett testfall (AI-drivet sällskapsspel) missade verktyget de mest relevanta, redan existerande konkurrenterna helt eftersom det bara resonerade från egen kunskap utan att söka.

2. **Nästa:** En fri "babbel"-startpunkt innan det guidade menyflödet — användaren skriver/pratar fritt, AI:n extraherar kandidat-riktningar som sedan bekräftas/förfinas via de styrda stegen. Design finns nu i `design-reference/kompass/`.

3. **Efter det:** Förgreningsfråga i början av flödet — "helt ny verksamhet" vs. "ny idé inom befintligt företag" — återanvänder samma motor, öppnar upp för befintliga företag som målgrupp utan att bygga ett separat läge.

4. **Egen produktfas, inte nu:** Sektionsindelad, återkommande "innovationsstöd"-version för befintliga företag som vill fortsätta strukturera nya idéer över tid (prenumerationsmodell). Byggs efter att grundflödet är validerat med riktiga användare, inte innan.

## Nästa konkreta steg

1. **Klart:** Lovable-koden är exporterad till GitHub — kodbasen Claude Code jobbar i är den exporten.
2. **Klart:** Research-abstraktionslagret + Perplexity-integration, med cachning, är byggt (se punkt 1 ovan).
3. Testa flödet på ytterligare 1–2 riktiga (inte egna) idéer efter Skärp-flödets ombyggnad, för att se om konkurrent-/evidensbilden blir korrekt i det nya flödet.
4. Lägg till fri babbel-startpunkt — design finns nu i `design-reference/kompass/` (konceptet "Kompassen lyssnar", byggt kring fritext först).
5. Utvärdera förgreningsfrågan för befintliga företag.

## Uttryckligen inte prioriterat just nu

- Automatisk dokumentation / delad vy mellan flera möten för rådgivare — annan typ av produkt (klienthantering), inte idéskärpning. Hålls som roadmap-notering, inte aktiv funktion.
- Full sektionsindelad dashboard-komplexitet för alla användare — hör hemma i produktfas 4 ovan, inte i MVP:t.
