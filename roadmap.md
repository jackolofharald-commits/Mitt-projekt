# Boggel – Roadmap

## Funktionsbacklogg, i prioritetsordning

1. **Klart:** Extern research (Perplexity Agent API) är inkopplad i kärnflödet, med cachning, bakgrundsstart och skydd mot dubbla/samtidiga anrop. I Skärp-läget (efter flödesombyggnaden, se `docs/skarp-flode.md`) används den i steg 4 ("Utvidga problemet") och steg 5–6 ("Välj problemram"/"Verkligheten") — motsvarande de gamla "problem"- och "konkurrenter"-stegen. I Kompass-läget (oförändrat flöde) fortfarande i problem- och konkurrentstegen.

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
