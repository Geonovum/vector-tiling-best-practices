# Best practices
**Werk in uitvoering**

## Wanneer vector tiles?
Welke afwegingen neem je, wanneer wel en wanneer beter niet vector tiles (beter iets anders)?
Sommige van de hieronder opgesomde redenen om vector tiles te gebruiken zijn ook van toepassing op andere vector data zoals bijvoorbeeld vectordata uit WFS of geojson. WFS en geojson hebben echter als belangrijk nadeel dat ze ofwel langzaam ofwel omvangrijk kunnen zijn waardoor problemen met datatransport en geheugen kunnen ontstaan. Het toepassen van tiles voor vector data lost de problemen van trage services of omvangrijke bestanden grotendeels op.

Een aantal redenen om vector tiles te gebruiken:
*   werken met omvangrijke vectordatasets
    *   alleen downloaden wat je nodig hebt
    *   minder geheugengebruik
    *   snellere weergave met minder rekentijd
*   met vector (-tiles) wordt de weergave bepaald door de weergavetoepassing
    *   selecteren en filteren van weer te geven data
    *   instellen van render parameters (kleur, lijndikte, transparantie, symbolen etc.) bij de data
    *   selecteren van weer te geven attributen, instellen klassificaties
    *   instellen tekenvolgorde
    *   on-the-fly koppelen met externe attribuut-data
*   met vector (-tiles) is de data aan de client-zijde beschikbaar voor meer interactieve toepassingen
    *   instantaan opvragen van data bij kaart-elementen
    *   interactief oplichten van elementen
    *   editen of verwijderen van elementen (=wegfilteren bronelement + toevoegen editable element)
    *   interactief filteren of selecteren van elementen
*   met vector (-tiles) kun je traploos zoomen
*   vector tiles zijn meestal compacter dan raster
    *   minder data versturen tussen client en server
    *   minder opslagruimte op server
*   vector tiles zijn zeer geschikt voor grootschalige toepassingen (hoge zoomlevels)
    *   mogelijkheid van overzooming (viewer verder ingezoomd dan zoomlevel van bron-tile)
    *   gedetailleerde tiles hoeven vanwege mogelijkheid overzoomen niet gegenereerd te worden
    *   bespaart rekentijd bij het voorbereiden van de data
    *   bespaart datatransport via het netwerk
    *   bespaart opslagruimte op de server
    *   door minder data meer efficiënt gebruik van caches

Redenen om geen vector tiles te gebruiken
*   weinig vector data (alles past in 1 bestand of in 1 download)
*   de brongegevens zijn rasterdata (foto's, dtms, andere data-rasters)
*   geen kennis, behoefte of tijd om eigen kaart-weergaves te maken
    *   (nog) geen standaardmethoden beschikbaar styles voor het renderen van vector uit te wisselen
*   er worden primitieve clients gebruikt die geen vector data kunnen renderen
*   het kost meer werk om gedetailleerde data te aggregeren voor kleinschalig (=lage zoomlevels) gebruik

## Aanbeveling: Optimalisatie tegel set

Tegels zo klein en clean mogelijk houden draagt bij aan de snelheid van het binnenhalen van de tegels en het renderen van de data aan de client side.

Wat kan je doen?

*   Goede pre-processing
*   Juiste configuratie data lagen in de tegel set
*   Optimale styling en bronnen binnen halen (zie A.3.2 uit Praktijkrichtlijn Vector Tiles)

### Pre-processing data

Simplificatie en nette geometrien maken voordat er vector tegels van worden gemaakt, helpt bij het klein houden van het tegel bestands formaat.

Door Simplificatie zo veel mogelijk in de voorbewerking van de data doen, houd je het meeste controle over hoe de data uiteindelijk gepresenteerd wordt.

Simplificatie kan op meerdere manieren behaald worden:

*   de vertices van de geometrien te beperken en te verminderen.
*   Geometrien daadwerkelijk versimpelen tot een simpelere vorm.
*   Kleinere coordinate precisie
*   Valide geometrien maken (Geen self-intersecting)
*   Aggregeren van data tot een grover niveau. Bijvoorbeeld aggregeren van pand naar blok of buurt niveau.

*   Verwijderen van snippets en heel kleine polygonen

*   attributen juist inrichten Bijvoorbeeld: minder decimalen achter de comma in de attribuut data of zelfs integer gebruiken

<div class="informative">
_AANBEVELING_ Aanbeveling doen rondom Simplificeren
Houdt ook rekening met bepaalde tussenpunten i.r.t. CRS.
</div>

### Tegel set Configuratie

*   Minder data op de lagere zoom niveaus laten zien. (aggregatie laten zien op lager zoom niveau)
*   Data op bepaalde zoom niveaus niet aanleveren.
*   Overzooming toestaan zodat de hoogste zoom niveaus geen tegels gerenderd hoeven worden.
*   Attribuut data beperken tot minimale vereiste voor de toepassing (niet gebruiken om alle data te laten zien die je in huis hebt)
*   Een geometrie soort per data laag gebruiken



### Aanbeveling: platte index
Bij tiling wordt normaal gesproken bij 1 niveau verder inzoomen een vector tile opgeknipt in 4 nieuwe vector tiles (een zogenaamde _platte index_ / _flat index_). Het kan nuttig zijn om vanwege datadichtheid dit niet te doen, maar slimmer om te gaan met bijna lege vector tiles (niet meer opknippen). Ook het overslaan van zoomniveaus wordt wel eens toegepast. Beide methodes vragen om een slimmere index, waarmee duidelijk is bij welke stappen wel of juist geen nieuwe tile opgevraagd moet worden. Een dergelijke slimme index is voor clients lastiger om te gebruiken. De Prakrijktrichtlijn schrijft daarom alleen een platte index voor.

<div class="advisement">
_AANBEVELING_ Gebruik een platte / "flat" index als indeling voor vector tiles.
</div>


### Aanbeveling: tile bestandsgrootte
<div class="informative">
_AANBEVELING_ Beperk de bestandsgrootte van een tegel, in omvang en hoeveelheid geometrie.bijvoorbeeld 500kb of 100.000 features
</div>

Sommige tooling bied mogelijkheden tot het configureren van simplificatie bij de generatie van vecotr tiles. Bijvoorbeeld:
*   weglaten van de kleinste polygonen
*   drop densest
*   drop fraction

Door de simplificatie aan de tooling over te laten krijg je makkelijker en snel kleinere tegels. Echter hierbij verlies je de invloed op de methode van simplificatie op jouw geometrien en kun je onverwachte resultaten krijgen. Daarom raden wij aan dat het beter is de data zelf eerst te optimaliseren om de meeste invloed op het simplificatie methode te hebben. En zo min mogelijk simplificatie door een genarator tool te laten doen.

## Aanbeveling: Clipping & buffering

<div class="informative">
_AANBEVELING_ Aanbeveling doen rondom clipping, ook rekening houden met CRS (o.a. mogelijk issue van lange lijnen i.c.m. herprojectie)
</div>


## Aanbeveling: Optimalisatie Styling en bron bevraging

Het aantal vector tile bronnen, lagen en de complexiteit van de features dragen allemaal bij aan de rendering tijd van de kaart.

Om een kaart sneller te maken kun je het aantal lagen, bronnen en de complexiteit van de feautres verminderen.

Meer tips zie: [https://docs.mapbox.com/help/troubleshooting/mapbox-gl-js-performance/](https://docs.mapbox.com/help/troubleshooting/mapbox-gl-js-performance/)


## Aanbeveling: afmetingen van de tile / high resolution tiles
Vector tiles bevatten informatie over de gebruikte coordinatenruimte, standaard 4096 volgens [[Mapbox-Vector-Tile-Specification]]. Dit betekent dat de ruimtelijke coordinaten omgerekend worden naar een intern systeem o.b.v. 4096 coordinaten breed en hoog.

De meeste vector tiles worden gemaakt voor gebruik in een client van 512 bij 512 pixels.

TODO: check van werkgroepleden voor een goede formulering
