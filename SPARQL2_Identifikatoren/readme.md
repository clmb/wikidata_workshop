
# Einführung in Identifikatoren 

Fokus:
  * Informationen, die von Wikidata verlinkt sind 

Zielsetzung:
  * Verstehen von Identifier und die Rolle von Wikidata als Authority Hub
  * Komplexere SPARQL-Abfragen mit Abfragen von externen Ressourcen


# Nutzung von Identifikationen auf Wikidata

Wir haben uns bisher nur mit dem „oberen“ Teil eines Datenobjekts  auseinandergesetzt, nämlich dem Teil, in dem das Datenobjekt durch Aussagen beschrieben wird. Daneben verweist aber Wikidata auch auf Beschreibungen zu diesen Datenobjekten, wie sie in anderen Datenbanken existieren, d.h. die Datenelemente in Wikidata sind mit externen Datenbanken vernetzt. Was sind die Vorteile …

**>>> FOLIE: Folie XX (Foliennummer: XX) <<<**

Lassen Sie uns dazu wieder zu unserem Beispiel zurückkehren:

  * Anzahl der Identifier bei Item Q762 (Leonardo da Vinci)

```sparql
SELECT (COUNT(?propertyclaim) AS ?count)
WHERE 
{
  wd:Q762 ?propertyclaim ?value  .
  ?property wikibase:directClaim ?propertyclaim .
  ?property wikibase:propertyType wikibase:ExternalId .
}
```
Das sind einige. Nun ist es aber für Sie wahrscheinlich wichtiger zu wissen, in welchem Umfang bestimmte Identifikatoren verwendet werden:

  * Gibt mir die Anzahl aller Items, die diesen Identifikator benutzen (wdt:P245 ULAN-ID):

```sparql 
SELECT (COUNT(?item) AS ?count) 
WHERE 
{
  ?item wdt:P245 ?id .
}
```

Das sind eher wenige... Ihr können Sie also noch sehr gut zu Wikidata beitragen :)

Wie können Sie sich aber nun diese Verweise zunutze machen?

Starten wir wieder mit dem bekannten Fall:

  * Gebe mir eine Liste von 100 Maler*innen mit ihrer deutschen Bezeichnung und ihrer ULAN-ID, falls vorhanden.

```sparql 
SELECT ?item ?itemLabel ?ulan 
WHERE 
{
  ?item wdt:P106 wd:Q1028181 .
  
  OPTIONAL { ?item wdt:P245 ?ulan. }
 
  SERVICE wikibase:label {
 	bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en" .
   }  
} LIMIT 100
```

Nun wollen wir aber auch den Link auf die ULAN-Referenz erhalten:

  * Gebe mir eine Liste von 100 Maler*innen mit ihrer deutschen Bezeichnung und ihrer ULAN-ID und dem externen Link.

```sparql 
SELECT ?item ?itemLabel ?ulan  ?url
WHERE 
{
  ?item wdt:P106 wd:Q1028181 .
  ?item wdt:P245 ?ulan .

  wd:P245 wdt:P1630 ?formatter .  
  BIND(IRI(REPLACE(?formatter, '\\$1', ?ulan)) AS ?url) .
 
  SERVICE wikibase:label {
 	bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en" .
   }  
} LIMIT 100
```

... und auch die RDF-Ressource, denn Sie wollen diese vielleicht in Ihrer persönlichen Datenbank nutzen.

  * Gebe mir eine Liste von 100 Maler*innen mit ihrer deutschen Bezeichnung und ihrer ULAN-ID und RDF.

```sparql 
SELECT ?item ?itemLabel ?ulan  ?url ?rdfResource
WHERE 
{
  ?item wdt:P106 wd:Q1028181 .
  ?item wdt:P245 ?ulan .
  
  wd:P245 wdt:P1630 ?formatter .  
  BIND(IRI(REPLACE(?formatter, '\\$1', ?ulan)) AS ?url) .
  
  wd:P245 wdt:P1921 ?rdfFormatter
  BIND(IRI(REPLACE(?rdfFormatter,'\\$1', ?ulan)) AS ?rdfResource) .

  SERVICE wikibase:label {
 	bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en" .
   }  
} LIMIT 100
```

Zum Ende möchte ich auf eine zentrale Idee von [Linked Open Data](https://de.wikipedia.org/wiki/Linked_Open_Data) eingehen. Auch der WDQS erlaubt es, sogenannte "federated queries" abzusetzen. Ich gebe Ihnen hier nur ein sehr einfaches Beispiel, aber ich denke, Sie erkennen die Möglichkeiten die darin liegen: 

  * Gib mir das Label und die Notizen zu Leonardo Da Vinci vom britischen Museum.

```sparql
PREFIX crm: <http://www.cidoc-crm.org/cidoc-crm/>
PREFIX rs: <http://www.researchspace.org/ontology/>
SELECT ?label ?note
WHERE 
{
  wd:Q762 wdt:P1711 ?id .
            
  BIND(URI(CONCAT("http://collection.britishmuseum.org/id/person-institution/", ?id)) AS ?bmID) .
             
  SERVICE <http://collection.britishmuseum.org/sparql> {
    ?bmID rs:displayLabel ?label;
          crm:P3_has_note ?note .      
  }
}
```

Das war es. Ich habe Ihnen nun einen ersten Einblick in die Möglichkeiten von Wikidata und der Abfrage der Daten gegeben. Aber ist wirklich nur ein erster Einstieg und es gibt noch viel mehr Möglichkeiten. 
