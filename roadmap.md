# Boggel – Roadmap

## Funktionsbacklogg, i prioritetsordning

1. **Nu:** Koppla in extern research (Perplexity API) i kärnflödet, framför allt i konkurrent- och problemstegen.

   Bevisat nödvändigt: i ett testfall (AI-drivet sällskapsspel) missade verktyget de mest relevanta, redan existerande konkurrenterna helt eftersom det bara resonerade från egen kunskap utan att söka.

2. **Nästa:** En fri "babbel"-startpunkt innan det guidade menyflödet — användaren skriver/pratar fritt, AI:n extraherar kandidat-riktningar som sedan bekräftas/förfinas via de styrda stegen. Design finns nu i `design-reference/kompass/`.

3. **Efter det:** Förgreningsfråga i början av flödet — "helt ny verksamhet" vs. "ny idé inom befintligt företag" — återanvänder samma motor, öppnar upp för befintliga företag som målgrupp utan att bygga ett separat läge.

4. **Egen produktfas, inte nu:** Sektionsindelad, återkommande "innovationsstöd"-version för befintliga företag som vill fortsätta strukturera nya idéer över tid (prenumerationsmodell). Byggs efter att grundflödet är validerat med riktiga användare, inte innan.

## Nästa konkreta steg

1. **Klart:** Lovable-koden är exporterad till GitHub — kodbasen Claude Code jobbar i är den exporten.
2. Bygg research-abstraktionslagret + Perplexity-integration i konkurrent- och problemstegen, med cachning.
3. Testa flödet på ytterligare 1–2 riktiga (inte egna) idéer efter research-kopplingen, för att se om konkurrentbilden nu blir korrekt.
4. Lägg till fri babbel-startpunkt — design finns nu i `design-reference/kompass/` (konceptet "Kompassen lyssnar", byggt kring fritext först).
5. Utvärdera förgreningsfrågan för befintliga företag.

## Uttryckligen inte prioriterat just nu

- Automatisk dokumentation / delad vy mellan flera möten för rådgivare — annan typ av produkt (klienthantering), inte idéskärpning. Hålls som roadmap-notering, inte aktiv funktion.
- Full sektionsindelad dashboard-komplexitet för alla användare — hör hemma i produktfas 4 ovan, inte i MVP:t.
