
# SPARQL Teil 2 – Grundlagen zu Identifikatoren 

Fokus:
  * Informationen, die von Wikidata verlinkt sind 

Zielsetzung:
  * Verstehen von Identifier und die Rolle von Wikidata als Authority Hub
  * Komplexere SPARQL-Abfragen mit Abfragen von externen Ressourcen

## Wikidata Datenmodell – Eigenschaften die Mitgliedschaft oder Teilhabe ausdrücken

Im Folgenden möchte ich noch einmal näher auf das Datenmodell eingehen. Wir haben bereits über Instanzen und Klassen gesprochen, aber ich habe Ihnen noch nicht erläutert, wie deren Modellierung in Wikidata erfolgt. Ich habe für meine Ausführungen vor allem diese Quelle verwendet: https://www.wikidata.org/wiki/Help:Basic_membership_properties/de 

In Wikidata gibt es Eigenschaften die Mitgliedschaft oder Teilhabe ausdrücken und diese sind durch die Eigenschaften „ist ein(e)“ (P31), „Unterklasse von“ (P279) und „ist Teil von“ (P361) präsentiert.

**>>> FOLIE: Eigenschaften die Mitgliedschaft oder Teilhabe ausdrücken (Foliennummer: XX) <<<**

Lassen Sie mich einige Beispiele geben:
  * `Klasse Mensch (Q5) mit den Instanzen Leonardo Da Vinci (Q762), Frida Kahlo (Q5588), Piet Mondrian (Q151803) etc.`
  * `Klasse Gemälde (Q3305213) mit den Instanzen Mona Lisa (Q12418), Selbstbildnis mit Affen (Q5712848), Broadway-Boogie-Woogie (Q2915406), etc.`
  * `Klasse Kunstmuseum (Q207694) mit den Instanzen Louvre (Q19675), Museum of Modern Art (Q188740), Bode-Museum (Q157825), etc.`

Gut, dass ist denke ich klar. Dann lassen Sie uns mal schauen, was wir damit machen können: 

  * Wie viele Instanzen der Klasse Mensch gibt es auf Wikidata? 

```sparql
SELECT (COUNT(?item) AS ?count)
WHERE
{
	?item wdt:P31 wd:Q3305213 .
}
```

Was ist neu in dieser Abfrage?

Ausdruck | Beschreibung
-------- | -------- 
COUNT | Es handelt sich um eine Aggregatfunktion, welche einfach die Anzahl der Ergebnisse umfasst.

Was bedeudet das, wenn wir Daten in Wikidata abfragen?

  *	Daten-Objekte (`Q...`) können entweder eine Instanz und/oder eine Klasse sein;
  * Mehrere Instanzen können in die Klasse A abstrahiert werden (wie in den obigen Beispielen);
  * Einige Klassen A können in Klasse B abstrahiert werden, die Beziehung zwischen A und B wird als Unterklasse von bezeichnet.

Was wir bereits wissen ist, dass "ist ein(e)“ (P31) die Beziehung zwischen Instanzen mit einem gemeinsamen Merkmal und einer Klasse, die durch dieses Merkmal gekennzeichnet ist, herstellt. „Unterklasse von" (P279) wird verwendet wenn alle Instanzen einer Klasse auch Instanzen einer anderen Klasse sind (`rdfs:subClassOf`). Wir verwenden "ist ein(e)" (P31) anstelle von "Unterklasse von" (P279), wenn wir über Instanzen mit einer solchen Beziehung nichts sagen können (`rdf:type`).

Schauen wir uns wieder einige Beispiele an:
  * `Kunstmuseum (Q207694) Unterklasse von (P279) Kulturmuseum (Q28737012)`
  * `Kulturmuseum (Q28737012) Unterklasse von (P279) Museum (Q33506)`
  * `Museum (Q33506) Unterklasse von (P279) Sammlung (Q2668072)`
  
Wenn Sie ihre Daten in Wikidata einfügen, ist das sehr wichtig, denn wenn bespielsweise ein Item A eine Instanz der Klasse B ist und Klasse B eine Unterklasse der Klasse C, dann ist Item A implizit auch eine Instanz der Klasse C. Es besteht keine generelle Notwendigkeit, eine Anweisung für die Relation A→C zu Wikidata hinzuzufügen (Transitivität).

O.k., soweit die Theorie, aber wie wirkt sich das auf unsere Abfragen aus? Schauen wir uns das im nachfolgenden Beispiel an:

* Wie viele Instanzen der Klasse Museum oder von Unterklassen von Museum gibt es auf Wikidata? 

```sparql
SELECT (COUNT(?item) AS ?count)
WHERE
{
	?item wdt:P31/wdt:P279* wd:Q33506 .
}
```

Und wieder ist etwas neu :)

Ausdruck | Beschreibung
-------- | -------- 
wdt:P31/wdt:P279* | Diese Darstellung (`/`) markiert einen "Property Path". Es handelt sich um eine Abkürzung, mit der wir definieren, dass das gesuchte `?item` "ist ein(e)" (P31) der Top-Klasse (Q33506) oder einer der Klassen in seinem Unterklassenbaum sein soll. Ein Sternchen (*) hinter einem Pfad-Element bedeutet "Null oder mehr von diesem Element".

Wahrscheinlich wird es klarer, wenn die gleiche Abfrage einfach nach bekannten Muster gestellt wird:

```sparql
SELECT (COUNT(?item) AS ?count)
WHERE
{
	?item wdt:P31 wd:Q33506 .
}
```

Sie sehen, die Anzahl ist unterschiedlich. 

Wer dieses Thema weiter vertiefen möchte sollte sich mit [Property Paths](https://en.wikibooks.org/wiki/SPARQL/Property_paths) beschäftigen. Es lohnt sich.

Und weil es so schön ist, gibt es hier gleich noch ein weiteres Beispiel:

  * Gib mir alle Museen in Deutschland (inklusive aller Unterklassen von Museen) und deren deutsche Bezeichnungen.

```sparql
SELECT DISTINCT ?museum ?museumLabel
WHERE 
{
  ?museum wdt:P31/wdt:P279* wd:Q33506 .
  ?museum wdt:P17 wd:Q183 .
        
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```

Wir haben hier unserer Abfrage das Schlüsselwort `DISTINCT` hinzugefügt, um doppelte Zählungen zu vermeiden (einfach mal die gleiche Abfrage ohne `DISTINCT` ausführen).

**>>> FOLIE: Folie mit praktischen Aspekten (Foliennummer: XX) <<<**

Nun kommen wir endlich zur letzten Eigenschaft, die ich an dieser Stelle noch einführen möchte "Teil von" (P361). Es kann sein, dass Datenobjekte keine Instanz einer anderen Instanz sind, aber sie können Teil einer anderen Instanz sein. Die Gemälde-Abteilung des Louvre (Q3044768) "ist Teil von" (P361) "Louvre" (Q19675). Auch Klassen können Teil einer anderen Klasse sein. "Kunst" (Q735) zum Beispiel "ist Teil von" (P361) "Kultur" (Q11042).

Versuchen Sie es wieder selbst:

  * Gebe mir alle Kunstsammlungen und deren deutsche Bezeichnungen, die Teil des Louvre sind.

```sparql
SELECT ?item ?itemLabel 
WHERE
{
   ?item wdt:P31 wd:Q7328910 .
   ?itemwdt:P361 wd:Q19675 .
  
   SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```

  * Gebe mir eine Liste alle Teile der Kultur und deren deutsche Bezeichnungen.

```sparql
SELECT ?item ?itemLabel 
WHERE
{
   ?item wdt:P361 wd:Q11042 .
  
   SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```

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
