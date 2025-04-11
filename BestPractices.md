# Best practices

## Wanneer vector tiles?

Welke afwegingen maak je bij het kiezen voor vector tiles of een andere oplossing? Sommige van de hieronder genoemde 
redenen om vector tiles te gebruiken, gelden ook voor andere vector data zoals WFS of geojson. WFS en geojson hebben 
echter als belangrijk nadeel dat ze traag of omvangrijk kunnen zijn, wat kan leiden tot problemen met datatransport 
en geheugen. Het gebruik van tiles voor vector data lost deze problemen grotendeels op.

Een aantal redenen om vector tiles te gebruiken:

* Met vectortiles wordt de weergave bepaald door de weergavetoepassing:
  * Data is aan de clientzijde beschikbaar voor meer interactieve toepassingen:
    * Direct opvragen van data bij kaart-elementen.
    * Interactief oplichten van elementen.
    * Bewerken of verwijderen van elementen (bronelement wegfilteren en bewerkbaar element toevoegen).
    * Interactief filteren of selecteren van elementen.
  * Vogelvluchtperspectief mogelijk (afhankelijk van de client).
* Werken met omvangrijke vectordatasets:
  * Serverside minder opslag.
  * Snellere weergave met minder rekentijd.
  * Alleen downloaden wat je nodig hebt.
* Met vector-tiles kun je traploos zoomen.
* Vector-tiles kunnen compacter zijn dan rasters:
  * Minder data versturen tussen client en server.
  * Minder opslagruimte op de server.
* Vector-tiles zijn zeer geschikt voor grootschalige toepassingen (hoge zoomlevels):
  * Mogelijkheid van overzooming (viewer verder ingezoomd dan zoomlevel van bron-tile).
  * Gedetailleerde tiles hoeven vanwege de mogelijkheid van overzooming niet gegenereerd te worden.
  * Bespaart rekentijd bij het voorbereiden van de data.
  * Bespaart datatransport via het netwerk.
  * Bespaart opslagruimte op de server.
  * Door minder data meer efficiënt gebruik van caches.

Redenen om geen vector tiles te gebruiken:
*   de brongegevens zijn rasterdata (foto's, DTMs, andere data-rasters)
*   er worden primitieve clients gebruikt die geen vector data kunnen renderen
*   voor lagere zoomlevels zijn veel resources nodig om te genereren, andere opties kunnen daar efficiënter zijn

## Aanbeveling: Optimalisatie tegel set

Het klein en schoon houden van tegels draagt bij aan de snelheid van het binnenhalen en renderen van de data aan de clientzijde.

Wat kun je doen?

* Goede pre-processing.
* Juiste configuratie van datalagen in de tegelset.
* Optimale styling en bronnen binnenhalen (zie A.3.2 uit Praktijkrichtlijn Vector Tiles).

### Pre-processing data

Simplificatie en nette geometrien maken voordat er vector tegels van worden gemaakt, helpt bij het klein houden van het tegel bestands formaat.

Door Simplificatie zo veel mogelijk in de voorbewerking van de data doen, houd je het meeste controle over hoe de data uiteindelijk gepresenteerd wordt.

Simplificatie kan op meerdere manieren behaald worden:

* detailnvieau vectoren laten matchen met het grid van de tegel of mogelijk iets grover (over het algemeen 4096 x 4096)  
* de vertices van de geometrien te beperken en te verminderen.
* Geometrien daadwerkelijk versimpelen tot een simpelere vorm.
* Aggregeren van data tot een grover niveau. Bijvoorbeeld aggregeren van pand naar blok of buurt niveau.
* per zoomlevel (of per groep zoomlevels) geometrieen versimpelen 
* een eenvoudiger geometrietype kiezen waar toepasbaar, bijvoorbeeld een lijn in plaats van een poygoon. 
*   Verwijderen van snippets en heel kleine polygonen
*   attributen klein houden door juiste datatype te kiezen; bijvoorbeeld: minder decimalen achter de komma in de attribuut data of

Hierbij is het belangrijk te realiseren dat vectortiles zijn geoptimaliseerd voor visualisatie. De orginele geometrieen
(de brondata) dienen op een andere manier verstrekt te worden.

Daarnaast zijn andere bewerkingen relevant: 
* Valide geometrieën maken (geen self-intersecting). Een vector in een vector tile is niet hetzelfde als een geometrie, dus zelfs met valide geometrieën kunnen nog steeds artefacten optreden.
* Attributen niet relevant voor visualisatie of feature info weglaten.
* Eventueel een verwijzing naar de brondata toevoegen aan de vectortile. 

<div class="informative">
_AANBEVELING_ Aanbeveling doen rondom Simplificeren
Houdt ook rekening met bepaalde tussenpunten i.r.t. CRS.
</div>

### Tegel set Configuratie

Bij het configureren van tegels op basis van geodata kunnen de volgende principes worden gehanteerd:
  
* Minder data op de lagere zoom niveaus laten zien. (aggregatie laten zien op lager zoom niveau)
*   Data op bepaalde zoom niveaus niet aanleveren.
*   Beperkt overzooming toepassen clientside, zodat de hoogste zoom niveaus geen tegels gerenderd hoeven worden.
*   Attribuut data beperken tot minimale vereiste voor de toepassing (niet gebruiken om alle data te laten zien die je in huis hebt)

### Aanbeveling: platte index
Bij tiling wordt normaal gesproken bij 1 niveau verder inzoomen een vector tile opgeknipt in 4 nieuwe vector tiles (een zogenaamde _platte index_ / _flat index_). Het kan nuttig zijn om vanwege datadichtheid dit niet te doen, maar slimmer om te gaan met bijna lege vector tiles (niet meer opknippen). Ook het overslaan van zoomniveaus wordt wel eens toegepast. Beide methodes vragen om een slimmere index, waarmee duidelijk is bij welke stappen wel of juist geen nieuwe tile opgevraagd moet worden. Een dergelijke slimme index is voor clients lastiger om te gebruiken. De Prakrijktrichtlijn schrijft daarom alleen een platte index voor.

<div class="advisement">
_AANBEVELING_ Gebruik een platte / "flat" index als indeling voor vector tiles.
</div>


### Aanbeveling: tile bestandsgrootte

Zie ook de praktijk richtlijn vectortiles. Het is mogelijk om eindeloos veel vectoren in een tegel op te nemen, met te hoog detailniveau. Hierbij kan de grootte van een vectortile ook eindeloos groot zijn. Hierom is extra aandacht nodig voor het beperken van de grootte van de tiles.  

Sommige tooling bied mogelijkheden tot het configureren van simplificatie bij de generatie van vecotr tiles. Bijvoorbeeld:
*   weglaten van de kleinste polygonen
*   drop densest
*   drop fraction

Door de simplificatie aan de tooling over te laten krijg je makkelijker en snel kleinere tegels. Echter hierbij verlies je de invloed op de methode van simplificatie op jouw geometrien en kun je onverwachte resultaten krijgen. Daarom raden wij aan dat het beter is de data zelf eerst te optimaliseren om de meeste invloed op het simplificatie methode te hebben. En zo min mogelijk simplificatie door een genarator tool te laten doen.

## Aanbeveling: Clipping & buffering

Veel tiling engines bieden de mogelijkheid om vectoren te clippen op de tegelrand. Daarbij wordt tevens een buffer toegpast
om styling van vectoren uit aangrenzende tegels vloeiend door te laten lopen. Als je dit niet doet worden de tegels groter 
dan noodzakelijk.

<div class="informative">
_AANBEVELING_ Maak waar mogelijk gebruik clipping van geometrieen, waarbij rekening gehoud met een buffer die 
groot genoeg is voor de toegepaste styling.  
</div>

## Aanbeveling: Optimalisatie Styling en bron bevraging

Het aantal vectortile bronnen, lagen en de complexiteit van de features dragen allemaal bij aan de rendering tijd van de kaart.

Om een kaart sneller te maken kun je het aantal lagen, bronnen en de complexiteit van de feautres verminderen.

Meer tips zie: [https://docs.mapbox.com/help/troubleshooting/mapbox-gl-js-performance/](https://docs.mapbox.com/help/troubleshooting/mapbox-gl-js-performance/)


## Aanbeveling: afmetingen van de tile / high resolution tiles

Vector tiles bevatten informatie over de gebruikte coordinatenruimte, standaard 4096 volgens [[Mapbox-Vector-Tile-Specification]]. Dit betekent dat de ruimtelijke coordinaten omgerekend worden naar een intern systeem o.b.v. 4096 coordinaten breed en hoog.

De meeste vector tiles worden gemaakt voor gebruik in een client van 256 bij 256 of 512 bij 512 pixels.


## Aanbeveling: Fonts

Verstrek lettertypen (fonts) zowel in Mapbox- als Webfont-formaat. Niet alle clients implementeren het Mapbox formaat. 