# Stroombeurs ⚡ handelsspel voor de day-ahead elektriciteitsmarkt

Een klassikaal multiplayer-spel: studenten handelen met hun telefoon in een day-ahead veiling met uniforme prijs (pay-as-clear). De docent projecteert de merit order, de marktprijs en de ranglijst.

- **Negen rollen:** kerncentrale, windpark, zonnepark, gascentrale, piekcentrale, batterij, energieleverancier, industrie (elektrolyser) en laadplein-exploitant.
- **Vijf speldagen van zes rondes.** Je kiest er één in de lobby:
  - **Standaard:** nacht, ochtendpiek, zonnige middag, windstille avondpiek, gasprijsschok en stormnacht.
  - **Winter (krapte):** hoge vraag door warmtepompen, duur gas, Dunkelflaute en een storing in de kerncentrale.
  - **Lente (overschot):** veel wind en zon bij lage vraag op een feestdag.
  - **Zomer (eendencurve):** subsidie voor wind en zon, negatieve prijzen op Pinksterzondag, een hittegolf die centrales beperkt, en een steile avondpiek.
  - **Herfst (onvoorspelbaar):** stormen en een gasprijs die schiet.
- Studenten hebben **geen account** nodig: ze scannen een QR-code en vullen een naam in.

Het spel is één statische pagina (`index.html`) op GitHub Pages. Firebase Realtime Database zorgt voor de live verbinding.

---

## Installatie (eenmalig, ±15 minuten)

### 1. Firebase-project aanmaken
1. Ga naar <https://console.firebase.google.com> en klik **Project toevoegen**. Google Analytics mag uit.
2. **Build → Authentication → Aan de slag → Sign-in method → Anoniem → Inschakelen.**
3. **Build → Realtime Database → Database maken.**
   - Kies locatie **Belgium (europe-west1)**.
   - Kies **Starten in vergrendelde modus**.
4. Open in de Realtime Database het tabblad **Regels**. Vervang de inhoud door die van `database.rules.json` en klik **Publiceren**.
5. **Projectinstellingen (tandwiel) → Algemeen → Je apps → Web (`</>`).** Registreer een app (Hosting hoeft niet). Kopieer het `firebaseConfig`-object.

### 2. Config invullen
Plak de waarden in `config.js`. Controleer dat **`databaseURL`** erbij staat. Die zie je bovenaan de Realtime Database-pagina.

De config is niet geheim en mag gewoon in een openbare repo. De beveiliging zit in de databaseregels:
- alleen de docent die een spel aanmaakt, kan het spel sturen;
- studenten kunnen alleen hun eigen bod indienen, en alleen zolang de markt open is;
- studenten kunnen elkaars biedingen niet lezen.

### 3. Op GitHub Pages zetten
1. Maak een nieuwe repository, bijvoorbeeld `stroombeurs`, en upload `index.html`, `config.js`, `database.rules.json` en `README.md`.
2. Ga naar **Settings → Pages → Build and deployment → Deploy from a branch**. Kies `main` en `/ (root)` en sla op.
3. Na ongeveer een minuut staat het spel op `https://<gebruikersnaam>.github.io/stroombeurs/`.
4. Lukt anoniem inloggen niet, voeg dan in Firebase via **Authentication → Settings → Authorized domains** het domein `<gebruikersnaam>.github.io` toe.

Lokaal testen kan ook. Draai `python3 -m http.server` in deze map en open <http://localhost:8000>. Openen als bestand (`file://`) werkt niet.

---

## In de les

1. Open de site op de laptop die aan de beamer hangt en klik **Nieuw spel starten**. Je krijgt een spelcode en een QR-code.
2. Studenten scannen de QR-code (of gaan naar de site en typen de code) en vullen een naam in. Ze krijgen automatisch een rol in een vaste volgorde, zodat vraag en aanbod in balans blijven. Tik op een rol om die te wijzigen.
3. Klik **Start de handelsdag**. Per ronde:
   - studenten dienen een volume en een minimum- of maximumprijs in;
   - jij ziet hoeveel biedingen binnen zijn, met een optionele timer van 60 of 90 seconden;
   - **Sluit markt en bereken** toont de merit order, de marktprijs en de ranglijst.
4. Na zes rondes volgen de eindstand, de beste speler per rol en het prijsverloop van de dag.

**Tips**
- **Prijsvoorspelling:** naast hun bod voorspellen studenten elke ronde de marktprijs. Hoe dichter bij de echte prijs, hoe meer punten (max 100 per ronde, naar verhouding van de prijs). Er is een aparte ranglijst voor de beste voorspellers. Ook spelers zonder vermogen in een ronde, zoals het zonnepark 's nachts, kunnen zo meedoen.
- **Nederlands / English:** rechtsboven staat een NL/EN-schakelaar. De keuze wordt per apparaat onthouden: het docentscherm kan Nederlands zijn terwijl een student in het Engels speelt.
- Studenten kunnen per ronde een **hint** openen met een tactiek voor hun rol in dat scenario. Met de knop **Hints aan/uit** op het docentscherm zet je ze voor iedereen uit, bijvoorbeeld in een tweede speelronde.
- Weinig studenten? Klik op **Aanvullen met bots**. Dan komt er alleen een bot bij voor elke rol die nog geen speler heeft. Een bot verwijder je met het kruisje.
- Zelf de studentenweergave bekijken: open de spellink in een privévenster of op je telefoon.
- Gebruik als docent steeds **dezelfde browser**. Firebase herkent je als eigenaar via een anonieme login die in die browser bewaard blijft. Op de startpagina staat een knop **Terug naar spel …**.

---

## Spelregels in het kort

| Rol | Kant | Kern van de rol |
|---|---|---|
| Kerncentrale | aanbod | 300 MW, €12/MWh; onder 150 MW kost €3.000 extra |
| Windpark | aanbod | 250 MW × windfactor, €0/MWh |
| Zonnepark | aanbod | 200 MW × zonfactor, €0/MWh |
| Gascentrale | aanbod | 400 MW, €85/MWh (stijgt mee met de gasprijs) |
| Piekcentrale | aanbod | 150 MW, €150/MWh (stijgt mee met de gasprijs) |
| Batterij | beide | 100 MW / 200 MWh, 90% laadrendement |
| Energieleverancier | vraag | Vaste klantvraag, verkoopt voor €140/MWh; tekort tegen onbalansprijs (≥ €300 of 2× de marktprijs) |
| Industrie | vraag | Tot 250 MW; stroom levert €120/MWh op |
| Laadplein-exploitant | vraag | Tot 150 MW per ronde, verkoopt voor €250/MWh; moet 450 MWh per dag laden, tekort kost €400/MWh |

**Winnaar:** de eindstand rangschikt op **% van verdienpotentieel**, niet op euro's, omdat rollen sterk verschillen in wat ze kunnen verdienen. Verdienpotentieel is wat je met jouw installatie had kunnen verdienen als je alle marktprijzen van de dag vooraf had gekend (als prijsnemer):
- **Producenten en industrie:** vol vermogen in elke ronde waarin dat winst oplevert.
- **Batterij:** de optimale laad- en ontlaadvolgorde.
- **Laadplein:** 450 MWh laden in de goedkoopste rondes.

Boven 100% betekent dat je de prijs in je voordeel hebt beïnvloed. Leveranciers worden gescoord op inkoopkosten (ideaal ÷ werkelijk, inclusief onbalans). Wie geen winstkans had en geen verlies leed, scoort 100%. Het eindscherm toont ook de beste per rol en het hoogste euroresultaat.

**Vereffening:** biedingen worden gesorteerd tot een aanbodcurve (oplopend) en een vraagcurve (aflopend). De marginale eenheid in het snijpunt bepaalt één prijs voor iedereen. Gelijke biedingen op de marge worden naar rato toegewezen. De prijsgrenzen zijn −€500 en €4.000/MWh.

Speldagen, kosten en rollen staan bovenaan het script in `index.html` (`SEASONS` en `ROLES`) en zijn makkelijk aan te passen. Een scenario kan naast vraag, wind, zon en gas ook `kern` (beschikbaarheid van de kerncentrale), `therm` (beschikbaarheid van gas- en piekcentrales, bijvoorbeeld bij warm koelwater) en `sub` (subsidie in €/MWh voor wind en zon) hebben. Bij de seizoensdagen worden de hints automatisch uit het scenario afgeleid.

## Onderhoud
- Elk spel staat in de database onder `games/<code>`. Oude spellen kun je in de Firebase-console verwijderen, onder `games` en `owners`.
- Het gratis Spark-abonnement van Firebase is ruim voldoende voor een klas.
