
# Einführung in Wissensmodellierung in Wikidata 

Fokus:
  * Zusammenhängen zwischen Datenelementen 

Zielsetzung:
  * Verstehen der Beziehungen zwischen den Datenelementen



## Eigenschaften, die Mitgliedschaft oder Teilhabe ausdrücken

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

Okay, dass soweit dazu, wenn Sie mehr wissen wollen, verweise ich Sie auf [diese Seite auf Wikidata](https://www.wikidata.org/wiki/Help:Basic_membership_properties/de).