
# SPARQL – Grundlagen und Einfache Abfragen 

  * Fokus:
    * Informationen die auf Wikidata direkt verfügbar sind 

  * Zielsetzung:
    * Verstehen des Aufbau der UI des WDQS 
    * Umsetzung einfacher SPARQL-Abfragen 
    * Anwendung der unterschiedlichen Visualisierungen im WDQS


## Einfache Abfragen

Gehe zu [Wikipedia-Seite von Leonardo_da_Vinci](https://de.wikipedia.org/wiki/Leonardo_da_Vinci). Nun gibt es in der linken Seite im Menü im Bereich *Werkzeuge* den Eintrag *Wikidata item*. Darüber gelangen Sie zu dem entsprechenden Wikidata-Eintrag https://www.wikidata.org/wiki/Q762. 

Was finden Sie auf dieser Seite?

Die Aussage *Leonardo da Vincis Werk ist Mona Lisa* besteht aus dem Subjekt *Leonardo da Vinci* (`Q762`), dem Prädikat *Werk ist* (`P800`) und einem Objekt *Mona Lisa* (`Q12418`). 

Diese Informationen müssen nun so umgewandelt werden, dass sie auch vom Rechner verstanden werden. Das Subjekt und das Prädikat (erster und zweiter Wert des Triple) müssen immer als URI gespeichert werden. Wenn das Subjekt zum Beispiel *Leonardo da Vinci* (`Q762`) ist, wird es als https://www.wikidata.org/wiki/Q762 gespeichert. Diese Aussage kann auch nun über die drei [https://en.wikipedia.org/wiki/Uniform_Resource_Identifier](URIs) dargestellt werden:

  * https://www.wikidata.org/wiki/Q762
  * https://www.wikidata.org/wiki/Property:P800
  * https://www.wikidata.org/wiki/Q12418

Präfixe erlauben es uns, diesen langen URI in kürzerer Form zu schreiben, um nun gleich unsere erste Query zu formulieren. Die URI  https://www.wikidata.org/wiki/Q762 kann im [WDQS](https://query.wikidata.org/) als `wd:Q762` abgekürzt werden. Im Gegensatz zu Subjekten und Prädikaten kann das Objekt (dritter Wert des Triple) entweder ein URI oder ein Literal (z.B. eine Zahl oder ein String) sein. 

Was können wir nun noch damit machen. Lassen Sie uns noch einmal die Wikidata-Seite zu Leonardo da Vinci anschauen: https://www.wikidata.org/wiki/Q762.

Wir sehen, dass *Leonardo da Vinci* (`Q762`) bestimmte *Werke* (`P800`) erstellt, unter anderem *Mona Lisa* (Q12418). Nun wollen wir aber wissen, welche Werke von Leonardo da Vinci neben der Mona Lisa erstellt wurden. Lassen Sie uns diese Suchanfrage in natürlicher Sprache auszudrücken: 

  * „Gib mir alle Werke von Leonardo Da Vinci.“ 

```sparql
SELECT ?item 
WHERE 
{
  wd:Q762 wdt:P800 ?item.
}
```
Lassen Sie uns die Query kurz zerlegen:

Ausdruck | Beschreibung
-------- | -------- 
`SELECT` | SELECT ist ein Ausdrucksmittel. Was ausgewählt und angezeigt wollen soll, wird später beschrieben. 
`?item`  | ?item ist nur die Platzhalter, genauer gesagt eine Variable. Eine Varible kann an dem vorangestellten Fragezeichen erkannt werden. Tip: Wähle selbsterklärende Variablen
`WHERE`  | Die WHERE-Klausel definiert, was in den Variablen steht. Eine WHERE-Klausel beginnt mit einer öffnenden geschweiften Klammer { und endet mit einer schließenden geschweiften Klammer }.
`wd:Q762`| Hier beginnt nun die eigentliche Aussage, mit dem Subjekt. Wir suchen nun alle Graphmuster, die die nachfolgende Struktur erfüllen. Einfach darüber hovern, dann sieht man, was es bedeutet. 
`wdt:P800` | Dies ist das Prädikat des Subjekts. Wenn Sie mit dem Mauszeiger darüber fahren, werden Sie sehen, dass es sich dabei um ein Beispiel handelt. 
`?item`  | Das ist das Objekt bzw. die Objekte, die wir suchen.

Was haben wir nun noch nicht erklärt haben sind die Abkürzungen:

WDQS versteht viele Abkürzungen bzw. Präfixe. Einige sind intern in Wikidata, z.B. `wd`, `wdt` (wdt = wikidatatruthy), `p`, `ps`, und viele andere sind häufig verwendete externe Präfixe, wie `rdf`, `skos`. Also noch mal:

  * PREFIX wd: =  http://www.wikidata.org/entity/
  * PREFIX wdt: = http://www.wikidata.org/prop/direct/

Glückwunsch, dass war nun die erste Abfrage. Nun ist aber das Ergebnis unserer Abfrage nicht sehr aussagekräftig, daher sollten wir auch die Bezeichnungen der Items dazu nehmen. Daher sollten wir unsere Aussage erweitern um:

  * „Gib mir von Leonardo Da Vinci alle Werke in eine Liste mit deutschen Bezeichnungen."

```sparql
SELECT ?item ?itemLabel 
WHERE 
{
     wd:Q762 wdt:P800 ?item.
     SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```

Was ist dazugekommen?

Ausdruck | Beschreibung
-------- | -------- 
?itemLabel | Wir können Bezeichnungen schneller erfassen als Zahlen. Ein Label ist der Name eines Artikels in einer Sprache, z.B. Englisch. 
SERVICE | Ein Service erweitert die Standardmöglichkeiten von SPARQL. Diese Services sind API-spezifisch.
SERVICE wikibase:label | Der Service ist sehr hilfreich, wenn Sie Labels abrufen möchten, da er die Komplexität von SPARQL-Abfragen reduziert, die Sie sonst benötigen würden, um den gleichen Effekt zu erzielen

**>>> FOLIE: Andere Services (Foliennummer ) <<<**

Ok. Es geht aber noch besser. Denn nun wollen Sie wissen:

  * „Gib mir alle Gemälde von Leonardo Da Vinci mit ihren Bezeichnungen, die in Florenz erstellt wurden."

```
SELECT ?item ?itemLabel 
WHERE
 {
  ?item wdt:P31 wd:Q3305213.
  ?item wdt:P1071 wd:Q2044.
  wd:Q762 wdt:P800 ?item.

  SERVICE wikibase:label{ bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```

Hinweis: Im WDQS gibt es eine Auto-Completion-Funktion, die mit `Ctrl+Space` aktiviert wird. 

Das geht auch kürzer:

```
SELECT ?item ?itemLabel
WHERE
 {
  ?item wdt:P31 wd:Q3305213; wdt:P1071 wd:Q2044.
  wd:Q762 wdt:P800 ?item.

  SERVICE wikibase:label{ bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```

Was ist nur an dieser Abfrage neu?

Ausdruck | Beschreibung
-------- | -------- 
`wdt:P31`  | „ist ein(e)“ 

Nun sind wir bereits mitten in der Datenmodellierung, denn diese Information hilft uns Beziehungen zwischen unterschiedlichen Konzepten herzustellen. Dazu müssen wir zwei zentralen Konzepte zu unterscheiden wissen:
  * Instanz: eine Einzelperson oder eine einzelne Sache
  * Klasse: 
    * Ein abstraktes Objekt, das Instanzen abstrahiert;
    * Jede Klasse ist durch eine Eigenschaft charakterisiert, die alle ihre Instanzen teilen;
    * Kann Instanzen mit anderen Klassen teilen, sich aber von anderen Klassen unterscheiden.

Die Beziehung zwischen Instanzen mit einem gemeinsamen Merkmal und einer Klasse, die durch dieses Merkmal gekennzeichnet ist, wird mit der Eigenschaft ist ein(e) (`P31`) hergestellt. 

Zum Beispiel, Mona Lisa (`Q12418`) und Der Schrei (`Q18891156`) sind beide Instanzen von Gemälde (`Q3305213`). Wir schreiben daher auf Wikidata:
  * Mona Lisa (Q12418) ist ein(e) (P31) Gemälde (Q3305213);
  * Der Schrei (Q18891156) ist ein(e) (P31) Gemälde (Q3305213).

Nun wissen Sie genug, um zu üben...

## Anwendung: Einfache Abfragen 

#### Aufgaben
**>>> FOLIE: "Anwendung Teil 1" (Foliennummer: ) <<<**

* Aufgabe 1: „Geben Sie mir eine Liste von allen Schüler*innen und deren deutsche oder englische Bezeichnungen von Leonardo Da Vinci”
* Aufgabe 2: „Geben Sie mir eine Liste von allen Gemälden und deren deutsche oder englische Bezeichnungen, welche zur Gemäldesammlung des Louvre gehören.”
* Aufgabe 3: „Geben Sie mir eine Liste von allen Gemälden und deren deutsche oder englische Bezeichnungen, zu deren Schlüsselereignissen ein Kunstraub gehört.”
* Aufgabe 4a (fortgeschritten): „Geben Sie mir eine Liste von allen Gemälden und deren deutsche oder englische Bezeichnungen, die inspiriert sind von einem Gemälde aus dem 16. Jahrhundert. Zusätzlich soll die deutsche oder englische Bezeichnung des ursprünglichen Gemäldes und das Jahr, wann dieses entstanden ist angezeigt werden. Die Liste soll nach der Jahreszahl sortiert werden”
* Aufgabe 4b (fortgeschritten): „Geben Sie mir eine Liste von allen Kunstmaler*innen und deren deutsche oder englische Bezeichnungen, deren Wirkungsort im Umkreis von 100 km um Rom  war.”


#### Lösungen
```
# Lösung: Aufgabe 1
SELECT ?item ?itemLabel 
WHERE 
{
  ?item wdt:P1066 wd:Q762.

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```

```sparql
# Lösung: Aufgabe 2
SELECT ?item ?itemLabel 
WHERE 
{
  ?item wdt:P31 wd:Q3305213.
  ?item wdt:P195 wd:Q3044768.
  
  SERVICE wikibase:label { bd:serviceParam wikibase:language “[AUTO_LANGUAGE],en”. }
}
```

```sparql
# Lösung: Aufgabe 3
SELECT ?item ?itemLabel 
WHERE 
{
  ?item wdt:P31 wd:Q3305213.
  ?item wdt:P793 wd:Q1756454.

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```

```sparql
# Lösung: Aufgabe 4a
SELECT ?item ?itemLabel ?paintingLabel (YEAR(?inception) AS ?year)
WHERE 
{  
  ?item wdt:P31 wd:Q3305213 .
  ?item wdt:P941 ?painting.
  ?painting wdt:P31 wd:Q3305213 .
  ?painting wdt:P571 ?inception .
  
  FILTER(1500 <= YEAR(?inception) && YEAR(?inception) < 1600)
  
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
} ORDER BY ?year
```

```sparql
# Lösung: Aufgabe 4b
SELECT ?item ?itemLabel ?placeLabel
WHERE 
{  
  ?item wdt:P106 wd:Q1028181 .
  ?item wdt:P937 ?place .
  ?place wdt:P17 wd:Q38 .
 
  wd:Q220 wdt:P625 ?romCoord .
  SERVICE wikibase:around { 
      ?place wdt:P625 ?coord . 
      bd:serviceParam wikibase:center ?romCoord . 
      bd:serviceParam wikibase:radius "100" . 
  }
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". } 
}
``` 

## Einfache Abfragen mit Datenvisualisierung 

Ein Aspekt auf den ich bisher nicht eingegangen bin, ist die Visualisierung der Daten, denn auch darin unterstützt Sie der WDQS. Beispielsweise gibt es in Wikidata eine Property (`P180`), welche zur Beschreibung des Inhalts der Bilder verwendet wird. Schauen wir uns ein Beispiel an. 

* „Gebe mir alle Gemälde, die einen Engel als Motiv haben.” (Ein anderes Beispiel wäre der "Horizont": wd:Q43261)

```sparql
SELECT ?item ?itemLabel ?inception
WHERE 
{
  ?item wdt:P31 wd:Q3305213.
  ?item wdt:P180 wd:Q235113.

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
} 
```

Die tabellarische Darstellung des Ergebnisses kennen Sie bereits. Nun wollen wir uns aber auch die Bilder anschauen. Diese Bilder werden automatisch aus [https://commons.wikimedia.org/wiki/Main_Page](Wikimedia Commons) geholt, über in Wikidata vorhandene Verlinkungen. Aber nun wäre es auch schön, die Bilder zu sehen.

Hinweis: Zeige die unterschiedliche Ergebnismenge erst ohne und dann mit OPTIONAL.

```sparql
SELECT ?item ?itemLabel ?image 
WHERE 
{
  ?item wdt:P31 wd:Q3305213.
  ?item wdt:P180 wd:Q235113.

  OPTIONAL { ?item wdt:P18 ?image. } (Ergänze später, um Image Grid zu zeigen.

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
} 
```

Was ist wieder dazugekommen:

Ausdruck | Beschreibung
-------- | -------- 
?image   | Eine Variable
OPTIONAL | Es handelt sich um ein neues Ausdrucksmittel. Es bedeutet, dass dieses Graphmuster nicht vorkommen muss, aber wenn es vorkommt, wird die Ergebnismenge um diese Information erweitert. OPTIONAL bezieht sich auf das rechts stehende Graphmuster ist aber linksassoziativ. 

Zeige nun die Visualisierung *ImageGrid*.

Welche Visualisierungen gibt es noch? Schauen wir mal, … , eine Timeline. Wie können wir diese nutzen? Wir müssen unsere Abfrage um die Zeit erweitern?  (Gehe auf ein Element in der Liste und zeige, wie die Zeit modelliert wurde)

```sparql
SELECT ?item ?itemLabel ?inception ?image 
WHERE 
{
  ?item wdt:P31 wd:Q3305213.
  ?item wdt:P180 wd:Q235113.
  ?item wdt:P571 ?inception.

  OPTIONAL { ?item wdt:P18 ?image. } (Ergänze später, um Image Grid zu zeigen.

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
} 
LIMIT 10 (Ergänze später, wenn Zeitstrahl gezeigt wird)
```

Nun muss ich hier immer klicken, um die richtige Visualsierung auszuwählen. Daher wäre es doch gut, wenn ich die Information zur Visualisierung gleich mitgeben würde:

```sparql
#defaultView:Timeline
SELECT ?item ?itemLabel ?inception ?image 
WHERE 
{
  ?item wdt:P31 wd:Q3305213.
  ?item wdt:P180 wd:Q235113.
  ?item wdt:P571 ?inception.

  OPTIONAL { ?item wdt:P18 ?image. } (Ergänze später, um Image Grid zu zeigen.

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
} 
LIMIT 10
```

**>>> FOLIE: "Mögliche Ansichten für die Ergebnisse" (Foliennummer: ) <<<**

Zeige wie diese Visualsierungen, auch innerhalb anderer Anwendugnen verwendet werden können (Export-Funktion des WDQS). 

### Anwendung: Einfache Abfragen mit Datenvisualisierung 

#### Aufgaben
**>>> FOLIE: "Anwendung Teil 2" (Foliennummer: ) <<<**

* Aufgabe 5: „Geben Sie mir eine Karte, in der alle Ort eingezeichnet sind, an denen sich aktuell Gemälde von Leonardo Da Vinci befinden.”
* Aufgabe 6 (fortgeschritten): „Gebe mir eine Karte auf der alle Museen und Unterklassen von Museen in Italien eingezeichnet sind, in denen sich Bilder befinden, dessen Urheber*innen zur Bewegung der Hochrenaissance gezählt werden. Sofern ein Bild und eine Website zu den Museen angegeben ist, sollen diese Informationen ebenfalls angezeigt werden ”



#### Lösungen
```sparql
# Lösung: Aufgabe 5
#defaultView:Map
SELECT ?item ?itemLabel ?coord 
WHERE 
{
  ?item wdt:P170 wd:Q762.
  ?item wdt:P276 ?place.
  ?place wdt:P625 ?coord.

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```

```sparql
# Lösung: Aufgabe 6
#defaultView:Map
SELECT DISTINCT ?museum ?museumLabel ?coord ?image ?website ?painter ?painterLabel 
WHERE 
{
  ?museum wdt:P31/wdt:P279* wd:Q33506 .
  ?museum wdt:P17 wd:Q38 .
  ?museum wdt:P625 ?coord .
  ?painting wdt:P276 ?museum .
  ?painting wdt:P170 ?painter .
  ?painter wdt:P135 wd:Q1474884 .
  
  OPTIONAL { ?museum wdt:P18 ?image . }
  OPTIONAL { ?museum wdt:P856 ?website . }
        
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
