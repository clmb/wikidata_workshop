
# SPARQL – Grundlagen und Einfache Abfragen 

Fokus:
  * Informationen, die auf Wikidata direkt verfügbar sind 

Zielsetzung:
  * Verstehen des Aufbaus der Benutzeroberfläche des Wikidata Query Service ([WDQS](https://query.wikidata.org/)) 
  * Eigenständiges Umsetzen einfacher SPARQL-Abfragen 
  * Anwendung der unterschiedlichen Visualisierungen im [WDQS](https://query.wikidata.org/)


## Einfache Abfragen

Gehen Sie zur [Wikipedia-Seite von Leonardo_da_Vinci](https://de.wikipedia.org/wiki/Leonardo_da_Vinci). Auf der linken Seite im Menü Unterpunkt *Werkzeuge* gibt es den Eintrag *Wikidata-Datenobjekt*. Darüber gelangen Sie zu dem entsprechenden Wikidata-Eintrag https://www.wikidata.org/wiki/Q762. 

Wie ist nun dieser Eintrag aufgebaut? Welche Informationen sind verfügbar?

Wir finden beispielsweise die Aussage "Leonardo da Vincis Werk ist Mona Lisa". Wir haben das Subjekt *Leonardo da Vinci* (`Q762`), mit dem Prädikat *Werk ist* (`P800`) und einem Objekt *Mona Lisa* (`Q12418`). (Die `Q...` werden sichtbar, wenn Sie über das entsprechende Element hovern.)

Um unsere erste Abfrage zu formulieren, müssen wir nun überlegen, in welche Form wir eine Aussage bringen müssen, damit sie auch von einem Rechner verstanden werden kann.

Das Subjekt und das Prädikat (erster und zweiter Wert des Triple) müssen immer als [URI](https://de.wikipedia.org/wiki/Uniform_Resource_Identifier) gespeichert werden. Wenn das Subjekt zum Beispiel *Leonardo da Vinci* (`Q762`) ist, wird es als https://www.wikidata.org/wiki/Q762 gespeichert. Damit kann die obige Aussage auch über die drei URI dargestellt werden:

```
https://www.wikidata.org/wiki/Q762   https://www.wikidata.org/wiki/Property:P800   https://www.wikidata.org/wiki/Q12418
```

Präfixe erlauben es uns, diese langen URI in kürzerer Form zu schreiben, um nun gleich unsere erste Query zu formulieren. Die URI  https://www.wikidata.org/wiki/Q762 kann im [WDQS](https://query.wikidata.org/) als `wd:Q762` abgekürzt werden. Im Gegensatz zu Subjekten und Prädikaten kann das Objekt (dritter Wert des Triple) entweder ein URI (im Wikidata Kontext ein Datenobjekt `Q...`) oder ein Literal (z.B. Zahl, String) sein. 

Wie können wir nun unser Wissen nutzen? 

Lassen Sie uns noch einmal die Wikidata-Seite zu [Leonardo da Vinci](https://www.wikidata.org/wiki/Q762) anschauen:

Wir wissen bereits, dass *Leonardo da Vinci* (`Q762`) bestimmte *Werke* (`P800`) erstellt, unter anderem *Mona Lisa* (Q12418). Nun wollen wir aber erfahren, welche Werke von Leonardo da Vinci neben der Mona Lisa erstellt wurden. Lassen Sie uns diese Suchanfrage in natürlicher Sprache auszudrücken: 

  * „Gib mir alle Werke von Leonardo Da Vinci.“ 

Diese Aussage "übersetzen" wir nun in eine erste Abfrage im [WDQS](https://query.wikidata.org/):

```sparql
SELECT ?item 
WHERE 
{
  wd:Q762 wdt:P800 ?item.
}
```
Hinweis: Im WDQS gibt es eine ["Auto-Completion-Funktion"](https://de.wikipedia.org/wiki/Autovervollst%C3%A4ndigung), die mit `Ctrl+Space` aktiviert wird. 

Lassen Sie uns die Query kurz zerlegen:

Ausdruck | Beschreibung
-------- | -------- 
`SELECT` | SELECT ist ein Ausdrucksmittel. Was ausgewählt und angezeigt wollen soll, wird später beschrieben. 
`?item`  | ?item ist eine Variable. Eine Varible kann an dem vorangestellten Fragezeichen erkannt werden. Tip: Wähle selbsterklärende Variablen. Diese Variable beschreibt, welchen Inhalt die Ergebnismenge umfasst. 
`WHERE`  | Die WHERE-Klausel definiert, wonach genau gesucht werden soll und was letztlich in den Variablen steht. Eine WHERE-Klausel beginnt mit einer öffnenden geschweiften Klammer { und endet mit einer schließenden geschweiften Klammer }.
`wd:Q762`| Hier beginnt die eigentliche Aussage, mit dem Subjekt. Einfach über das Element hovern, dann sehen sie die Bezeichnung. Das Präfix `wd:` kürzt sie URI http://www.wikidata.org/entity/ ab. 
`wdt:P800` | Dies ist das Prädikat. Das Präfix `wdt:` repräsentiert die URI http://www.wikidata.org/prop/direct/. (wdt = wikidatatruthy)
`?item`  | Das ist das Objekt bzw. die Objekte, die wir suchen. In diesem Fall alle Werke, die 

Glückwunsch, dass war nun ihre erste Abfrage. Wenn wir uns das Ergebnis unserer Abfrage anschauen (eine Liste mit Q-Nummern), ist es nicht sehr aussagekräftig, daher sollten wir auch die Bezeichnungen der Items dazu nehmen. Lassen Sie uns also unsere Aussage erweitern um:

  * „Gib mir alle Werke von Leonardo Da Vinci mit den deutschen Bezeichnungen."

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
?itemLabel | Wir können Bezeichnungen schneller erfassen als Zahlen. Eine Bezeichnung (Label) ist der Name eines Datenobjekts (Q...) in einer Sprache, z.B. Englisch. 
SERVICE | Ein Service erweitert die Standardmöglichkeiten von SPARQL im WDQS und macht es einfacher, bestimmte Abfragen zu stellen. Diese Services sind API-spezifisch und können nicht in anderen Endpoints verwendet werden.
SERVICE wikibase:label | Dieser Service ist sehr hilfreich, wenn Sie Bezeichnungen abrufen möchten. "AUTO_LANGUAGE" bedeudet, dass die Bezeichnung des Datenobjekts in der Sprache der Benutzeroberfläche (in unserem Fall deutsch) angezeigt wird. Falls keine Bezeichnung in deutscher Sprache existiert, wird auf die englische Bezeichnung zurückgegriffen (daher "en").

**>>> FOLIE: Andere Services (Foliennummer: XX) <<<**

Ok. Es geht aber noch besser. Denn nun wollen Sie wissen:

  * „Gib mir alle Gemälde von Leonardo Da Vinci mit ihren Bezeichnungen, die in Florenz erstellt wurden."

```sparql
SELECT ?item ?itemLabel 
WHERE
 {
  ?item wdt:P31 wd:Q3305213.
  ?item wdt:P1071 wd:Q2044.
  wd:Q762 wdt:P800 ?item.

  SERVICE wikibase:label{ bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
Das geht auch syntaktisch kürzer. Sie können die erste und zweite Aussage zusammenziehen, da sich beide auf das gleiche Subjekt beziehen und durch ein Semikolon trennen (Kommatrennung existiert, wenn zwei oder mehr Aussagen das gleiche Objekt suchen).

```sparql
SELECT ?item ?itemLabel
WHERE
 {
  ?item wdt:P31 wd:Q3305213; wdt:P1071 wd:Q2044.
  wd:Q762 wdt:P800 ?item.

  SERVICE wikibase:label{ bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
Ich werde aber in diesem Tutorial nicht mit der verkürzten Schreibweise arbeiten, um die Lesbarkeit der Beispiele zu erhöhen. 

Was ist nun an dieser Abfrage inhaltlich neu?

Ausdruck | Beschreibung
-------- | -------- 
`wdt:P31`  | „ist ein(e)“ 

Nun sind wir bereits mitten in der Datenmodellierung, denn diese Information hilft uns, Beziehungen zwischen unterschiedlichen Konzepten (Datenobjekten) herzustellen. Dazu müssen wir zwei zentrale Konzepte unterscheiden:
  * Instanz: 
    * eine Einzelperson 
    * eine einzelne Sache
  * Klasse: 
    * ein abstraktes Objekt, das Instanzen abstrahiert
    * jede Klasse ist durch eine Eigenschaft charakterisiert, die alle ihre Instanzen teilen
    * kann Instanzen mit anderen Klassen teilen, sich aber von anderen Klassen unterscheiden.

Die Beziehung zwischen Instanzen mit einem gemeinsamen Merkmal und einer Klasse, die durch dieses Merkmal gekennzeichnet ist, wird mit der Eigenschaft *ist ein(e)* (`P31`) hergestellt. 

Zum Beispiel, *Mona Lisa* (`Q12418`) und *Der Schrei* (`Q18891156`) sind beides Instanzen von *Gemälde* (`Q3305213`). Daher finden wir auf Wikidata folgende Aussagen:
  * Mona Lisa (Q12418) ist ein(e) (P31) Gemälde (Q3305213);
  * Der Schrei (Q18891156) ist ein(e) (P31) Gemälde (Q3305213).

Nun wissen Sie genug, damit Sie selbst üben können...

## Anwendung: Einfache Abfragen 

Diese Aufgaben sollten möglichst selbstständig gelöst werden. 

### Aufgaben
**>>> FOLIE: "Anwendung Teil 1" (Foliennummer: ) <<<**

* Aufgabe 1: „Geben Sie mir eine Liste von allen Schüler*innen und deren deutsche oder englische Bezeichnungen von Leonardo Da Vinci”
* Aufgabe 2: „Geben Sie mir eine Liste von allen Gemälden und deren deutsche oder englische Bezeichnungen, welche zur Gemäldesammlung des Louvre gehören.”
* Aufgabe 3: „Geben Sie mir eine Liste von allen Gemälden und deren deutsche oder englische Bezeichnungen, zu deren Schlüsselereignissen ein Kunstraub gehört.”
* Aufgabe 4a (fortgeschritten): „Geben Sie mir eine Liste von allen Gemälden und deren deutsche oder englische Bezeichnungen, die inspiriert sind von einem Gemälde aus dem 16. Jahrhundert. Zusätzlich soll die deutsche oder englische Bezeichnung des ursprünglichen Gemäldes und das Jahr, wann dieses entstanden ist angezeigt werden. Die Liste soll nach der Jahreszahl sortiert werden”
* Aufgabe 4b (fortgeschritten): „Geben Sie mir eine Liste von allen Kunstmaler*innen und deren deutsche oder englische Bezeichnungen, deren Wirkungsort im Umkreis von 100 km um Rom  war.”

### Lösungen
```sparql
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

Ein Aspekt auf den ich bisher nicht eingegangen bin, ist die Visualisierung der Daten, denn auch darin unterstützt Sie der WDQS. Beispielsweise gibt es in Wikidata eine Eigenschaft (`P180`), welche zur Beschreibung des Inhalts der Bilder verwendet werden kann. Schauen wir uns ein Beispiel an. 

* „Gebe mir alle Gemälde, die einen Engel als Motiv haben.” (Ein anderes Beispiel wäre der "Horizont": wd:Q43261)

```sparql
SELECT ?item ?itemLabel
WHERE 
{
  ?item wdt:P31 wd:Q3305213.
  ?item wdt:P180 wd:Q235113.

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
} 
```

Die tabellarische Darstellung des Ergebnisses kennen Sie bereits. Nun wollen wir diese Bilder auch sehen. Diese Bilder werden automatisch aus [Wikimedia Commons](https://commons.wikimedia.org/wiki/Main_Page) geholt, über in Wikidata vorhandene Verlinkungen. 

Dazu mussen wir die Query erweitern...

```sparql
SELECT ?item ?itemLabel ?image 
WHERE 
{
  ?item wdt:P31 wd:Q3305213.
  ?item wdt:P180 wd:Q235113.

  OPTIONAL { ?item wdt:P18 ?image. } 

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
} 
```
Bitte prüfen Sie die unterschiedliche Ergebnismenge einmal ohne OPTIONAL, d.h. `?item wdt:P18 ?image.` dann mit `OPTIONAL {?item wdt:P18 ?image.}`. Was fällt Ihnen auf?

Aber lassen Sie nun schauen, wie die Abfrage erweitert wurde:

Ausdruck | Beschreibung
-------- | -------- 
?image   | Eine Variable
OPTIONAL | Es handelt sich um ein neues Ausdrucksmittel. Es bedeutet, dass diese Aussage auf den entsprechenden Objekten nicht vorkommen muss, aber wenn es vorkommt, wird die Ergebnismenge um diese Information erweitert. OPTIONAL bezieht sich auf die rechts stehende Aussage ist aber linksassoziativ. 

Wählen Sie nun im Drop-Down Menü die Visualisierung *ImageGrid* aus.

Welche Visualisierungen gibt es noch? Schauen wir mal, … , eine Timeline. Wie können wir diese nutzen? Wir müssen unsere Abfrage um die Zeit erweitern?  (Gehe auf ein Element in der Liste und zeige, wie die Zeit in dem Datenobjekt modelliert wurde.)

```sparql
SELECT ?item ?itemLabel ?inception ?image 
WHERE 
{
  ?item wdt:P31 wd:Q3305213.
  ?item wdt:P180 wd:Q235113.
  ?item wdt:P571 ?inception.

  OPTIONAL { ?item wdt:P18 ?image. } 

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
} 
LIMIT 10 
```

Das `LIMIT 10` habe ich eingefügt, um meine Ergebnismenge auf 10 Datenelement zu reduzieren, um damit die Lesbarkeit zu erhöhen.

Nun muss ich immer das Dropdown-Menü benutzen, um eine Visualsierung auszuwählen. Es wäre doch gut, wenn ich die Information zur Visualisierung meiner Abfrage gleich mitgeben könnte:

```sparql
#defaultView:Timeline
SELECT ?item ?itemLabel ?inception ?image 
WHERE 
{
  ?item wdt:P31 wd:Q3305213.
  ?item wdt:P180 wd:Q235113.
  ?item wdt:P571 ?inception.

  OPTIONAL { ?item wdt:P18 ?image. } 

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
