# Kapitel 5: Komponieren durch Programmieren

## Functional husbandr
## Funktionale Landwirtschaft

Hier kommt `compose`:

```js
var compose = function(f,g) {
  return function(x) {
    return f(g(x));
  };
};
```

`f` und `g` sind Funktionen während `x` der Wert ist der durch sie hindurch "geschleust" wird.

Composition fühlt sich wie Funktionen Landwirtschaft an. Du, der Züchter von Funktionen, wählst zwei mit Eigenschaften, die Du kombinieren möchtest und mischt sie zusammen um eine neue völlig neue zu erzeugen.

```js
var toUpperCase = function(x) { return x.toUpperCase(); };
var exclaim = function(x) { return x + '!'; };
var shout = compose(exclaim, toUpperCase);

shout("send in the clowns");
//=> "SEND IN THE CLOWNS!"
```

Die Komposition zweier Funktionen liefert eine neue Funktion zurück. Das macht Sinn: Zwei Einheiten eines bestimmten Typs zu kombinieren (in diesem Fall Funktionen) sollte eine neue Einheit des selben Typs ergeben. Man steckt keine zwei Legos zusammen und bekommt einen Holzklotz. Dahinter verbirgt sich eine Theorie, die wir bald entdecken werden.

In unserer Definition von `compose`, wird `g` vor `f` ausgeführt, was einen rechts links Datenfluß ergibt. Dies ist viel lesbarer als das verschachteln von Funktionsaufrufen. Ohne compose, müssten wir für oben schreiben:

```js
var shout = function(x){
  return exclaim(toUpperCase(x));
};
```

Anstatt von innen nach außen auszuführen, führen wir von rechts nach links aus. Hier ist ein Beispiel, wo es auf die Reihenfolge ankommt:

```js
var head = function(x) { return x[0]; };
var reverse = reduce(function(acc, x){ return [x].concat(acc); }, []);
var last = compose(head, reverse);

last(['jumpkick', 'roundhouse', 'uppercut']);
//=> 'uppercut'
```

`reverse` dreht die Liste um und `head` schnappt sich das erste Element. Daraus resultiert praktisch eine wenn auch ineffiziente `last` Funktion. Die Reihenfolge der Funktionen in der Komposition sollte offensichtlich sein. Wir könnten eine links-rechts Version definieren, aber wir kommen der mathematischen Version so wesentlich näher. Es stimmt, Komposition entstammt direkt den Mathebüchern. Vielleicht ist es schon an der Zeit, sich die Eigenschaften jeder Komposition anzusehen.

```js
// Assoziativ Gesetz
var associative = compose(f, compose(g, h)) == compose(compose(f, g), h);
// trifft zu
```

Komposition ist assoziativ, was heißt, dass es keine Rolle spielt wie man sie gruppiert. Wenn wir also den String in Großbuchstaben ausgeben wollen, können wir schreiben:

```js
compose(toUpperCase, compose(head, reverse));

// oder
compose(compose(toUpperCase, head), reverse);
```

Da es keinen Unterschied macht wie wir die Aufrufe gruppieren, die wir `compose` übergeben, wird das Ergebnis das gleiche sein. Das gibt uns die Möglichkeit, compose als variadische Funktion zu schreiben, die wir so verwenden können:

```js
// zuvor mussten wir zweimal compose schreiben, aber weil es assoziativ ist, können wir compose so viele fn's übergeben wie wir wollen und es selbst entscheiden lassen, wie sie zu gruppieren sind.
var lastUpper = compose(toUpperCase, head, reverse);

lastUpper(['jumpkick', 'roundhouse', 'uppercut']);
//=> 'UPPERCUT'


var loudLastUpper = compose(exclaim, toUpperCase, head, reverse)

loudLastUpper(['jumpkick', 'roundhouse', 'uppercut']);
//=> 'UPPERCUT!'
```

Das Anwenden des Assoziativ Gesetzes gibt uns Flexibilität und erhält unseren Selenfrieden weil das Ergebnis das Gleiche sein wird. Die etwas kompliziertere variadische Definition ist teil der mitgelieferten Libs für dieses Buch und ist die übliche Definition, die man in Libraries wie [lodash][lodash-website], [underscore][underscore-website], und [ramda][ramda-website] findet.

Ein netter Nebeneffekt des Assiziativ Gesetzes ist, dass wir jede Gruppe von Funktionen, die wir extrahieren und in ihrer eigenen Komposition bündeln können. Jetzt wollen wir mal ein bisschen mit unserem vorherigen Beispiel herumspielen:

```js
var loudLastUpper = compose(exclaim, toUpperCase, head, reverse);

// oder
var last = compose(head, reverse);
var loudLastUpper = compose(exclaim, toUpperCase, last);

// oder
var last = compose(head, reverse);
var angry = compose(exclaim, toUpperCase);
var loudLastUpper = compose(angry, last);

// mehr Variationen...
```

Es gibt hier kein Richtig oder Falsch - wir stecken einfach unsere Legos zusammen wie es uns gefällt. Normalerweise gruppiert man Funktionen auf wieder verwendbare Weise wie `last` und `angry`. Wenn Du Fowlers's "[Refactoring][refactoring-book]" kennst, wirst Du  "[extract method][extract-method-refactor]" hier wieder erkennen ...allerdings ohne Dich um den Objektstatus kümmern zu müssen.

## Pointfree

Pointfree Style bedeutet nie seine Daten nennen zu müssen. Entschuldigung. Es meint Funktionen, die nie die Daten aufzählen müssen, auf denen sie operieren. First Class Functions, Currying und Komposition harmonieren in diesem Style.

```js
//nicht Pointfree, weil wir die Daten erwähenen: word
var snakeCase = function (word) {
  return word.toLowerCase().replace(/\s+/ig, '_');
};

//Pointfree
var snakeCase = compose(replace(/\s+/ig, '_'), toLowerCase);
```

Hast Du gesehen wie wir `replace` teilweise aplliziert haben? Was wir machen ist unsere Daten durch Funktionen mit je einem Argument zu schleusen. Currying erlaubt es uns, unsere Funktionen so vorzubereiten, dass sie ihre Daten entgegen nehmen, damit arbeiten und sie weiter leiten. Ist Dir aufgefallen, das wir die Daten nicht nennen mussten um unsere Funktion in der Pointfree Version zu konstruieren? Nicht wie in der Pointful Variante, in vor allem Anderen `word` benennen mussten.

Hier noch ein weiteres Beispiel.

```js
//nicht Pointfree weil wir eine Variable benennen: name
var initials = function (name) {
  return name.split(' ').map(compose(toUpperCase, head)).join('. ');
};

//pointfree
var initials = compose(join('. '), map(compose(toUpperCase, head)), split(' '));

initials("hunter stockton thompson");
// 'H. S. T'
```

Nochmal; Pointfree Code kann uns helfen unnötige Namen wegzulassen und doch prägnant und generisch zu bleiben. Pointfree ist ein guter Lackmustest für Funktionalen Code, da es uns zeigt ob wir kleine Funktionen haben, die Input nach Output transferieren. Man kann z.B. keine while-Schleife komponieren. Allerdings ist Pointfree auch eine zweischneidige Klinge und kann die Intention verschleiern. Nicht jeder Funktionaler Code ist Pointfree und das ist O.K. so. Wir verfolgen ihn, andernfalls bleiben wir bei normalen Funktionen.

## Debugging
Ein verbreiteter Fehler ist etwas wie `map` mit einer Funktion zu kombinieren, die zwei Argumente erwartet, ohne dass wir sie vorher teilweise appliziert hätten.

```js
//falsch - wir übergeben angry ein Array und applezieren map Gott weiß was.
var latin = compose(map, angry, reverse);

latin(["frog", "eyes"]);
// Fehler


// richtig - jede Funktion erwartet ein Argument.
var latin = compose(map(angry), reverse);

latin(["frog", "eyes"]);
// ["EYES!", "FROG!"])
```

Wenn Du Probleme hast eine Komposition zu debuggen, kannst Du diese hilfreiche trace Funktion verwenden, die impure ist, um zu sehen was vor sich geht.

```js
var trace = curry(function(tag, x){
  console.log(tag, x);
  return x;
});

var dasherize = compose(join('-'), toLower, split(' '), replace(/\s{2,}/ig, ' '));

dasherize('The world is a vampire');
// TypeError: Cannot read property 'apply' of undefined
```

Irgendwas ist hier faul. Fragen wir `trace`

```js
var dasherize = compose(join('-'), toLower, trace("after split"), split(' '), replace(/\s{2,}/ig, ' '));
// after split [ 'The', 'world', 'is', 'a', 'vampire' ]
```

Ach so!, Wir müssen `toLower` in `map` stecken, weil es mit einem Array arbeitet.

```js
var dasherize = compose(join('-'), map(toLower), split(' '), replace(/\s{2,}/ig, ' '));

dasherize('The world is a vampire');

// 'the-world-is-a-vampire'
```

Die `trace` Funktion erlaubt uns Daten an einem bestimmten Punkt für debugging Zwecke einzusehen. Sprachen wie Haskell und Purescript haben ähnliche Funktionen um die die Entwicklung zu vereinfachen.

Komposition wird unser Werkzeug um Programme zu kontruieren und, wie es der Zufall so will, wird von einer mächtigen Theorie gestützt, die sicher stellt, dass alles in unserem Sinne verläuft. Wie lautet diese Theorie?


## Kategorientheorie

Kategorientheorie ist ein abstrakter Zweig der Mathematik und formalisiert Konzepte aus verschiedenen anderen Zweigen wie der Mengenlehre, Typentheorie, Gruppentheorie, Logik und vieles mehr. Sie arbeitet hauptsächlich mit Objekten, Morphismen und Transformationen. Also in etwa das was Programmierung ausmacht. Hier ist eine Übersicht der selben Konzepte aus dem Blickwinkel jeder Theorie.

/* Bild Übersetzen */
<img src="images/cat_theory.png" />

Sorry, ich wollte Dich nicht erschrecken. Ich erwarte nicht, dass Du Dich schon genau mit diesen Konzepten auskennst. Aber ich möchte aufzeigen, wieviel Überschneidung es hier gibt und warum Kategorientheorie darauf abzielt, diese Dinge zu vereinen.

In der Kategorientheorie haben wir etwas, das sich ... Kategorie nennt. Es ist definiert als eine Sammlung mit folgenden Komponenten:

  * Eine Objektsammlung
  * Eine Morphismensammlung
  * Den Begriff der Komposition von Morphismen
  * Ein herausgehobener Morphismus namens identity

Category theory is abstract enough to model many things, but let's apply this to types and functions, which is what we care about at the moment.
Kategorientheorie ist abstrakt genug um viele Dinge zu modellieren, aber wir wollen sie auf Typen und Funktionen anwenden, was uns momentan am meisten interessiert.

**Eine Objektsammlung**
Die Objekte werden Datentypen sein. Z.B. ``String``, ``Boolean``, ``Number``, ``Object``, etc. Wir betrachten Datentypen oft als eine Menge all der möglichen Werte. Man könnte ``Boolean`` als eine Menge von `[true, false]` betrachten und ``Number`` als eine Menge aller möglichen Zahlenwerte. Typen wie Mengen zu behandeln ist sinnvoll, denn wir können die Mengenlehre nutzen um mit ihnen zu arbeiten.

**Morphismensammlung**
Die Morphismen sind unsere stinknormalen PureFunctions.

**Den Begriff der Komposition von Morphismen**
Du wirst es erraten haben: Das ist unser brand neues Spielzeug - `compose`. Wir haben darüber gesprochen, dass unsere `compose` Funktion assoziatv ist. Das ist kein Zufall, denn in der Kategorientheorie muss jede Komposition dieser Eigenschaft gerecht werden.

Das folgende Bild demonstriert die Komposition:

<img src="images/cat_comp1.png" />
<img src="images/cat_comp2.png" />

Hier ein konkretes Beispiel als Code:

```js
var g = function(x){ return x.length; };
var f = function(x){ return x === 4; };
var isFourLetterWord = compose(f, g);
```

**Ein  herausgehobener Morphismus namens identity**
Lass uns eine nützliche Funktion namens `id` einführen. Diese Funktion nimmer einfach Input und spuckt ihn Dir wieder zurück. Schau mal:

```js
var id = function(x){ return x; };
```

Du wirst Dich fragen "Wofür zum Teufel soll das denn gut sein?". In den folgenden Kapiteln werden wir diese Funktion noch häufig benutzen. Aber vorerst stell Dir vor, dass diese Funktion ein Platzhalter für unseren Wert ist - eine Funktion die so tut als wäre sie ein allerwelts Wert.

`id` passt sehr gut zu compose. Hier ist eine Eigenschaft, die auf jede Unary Function f (eine Funktion mit nur einem Argument):

```js
// identity
compose(id, f) == compose(f, id) == f;
// true
```

Hey, das ist wie die Identitätseigenschaft bei Zahlen! Wenn das nicht sofort klar wir, nimm Dir Zeit dafür. Verstehe die Sinnlosigkeit. Wir werden `id` bald über all sehen, für jetzt betrachte sie als einen Platzhalter für einen gegebenen Wert. Das sit sehr nützlich, wenn man Pointfree Code schreibt.

Da hast Du es also, eine Kategorie von Typen und Funktionen. Wenn das deine erste Berührung mit dem Thema ist, nehme ich an, wird Dir noch nicht ganz klar sein, was eine Kategorie ist un warum sie nützlich ist. Auf diesem Wissen werden wir das restliche Buch hindurch aufbauen. Ab jetzt, in diesem Kapitel, in dieser Zeile, wirst Du mindestens verstehen, dass sie uns eine gewisse Weisheit bezüglich Komposition verleiht, nämlich das Assoziativ Gesetz und das Neutrale Element.

Ob es andere Kategorien gibt, fragst Du? Nun, wir könnten eine für gerichtete Graphen definieren, in der Knoten Objekte, Kanten Morphismen und Komposition schlicht Pfadverkettung. Wir können Zahlen als Objekte definieren und `>=` als Morphismen (eigentlich kann jede teilweise vollständige Reihenfolge eine Kategorie sein). Es gibt haufenweise Kategorien, aber was dieses Buch angeht, werden wir uns mit den oben genannten begnügen. Wir haben genug an der Oberflache gekratzt und müssen jetzt weiter kommen.


## Zusammenfassung
Komposition verbindet unsere Funktionen wie ein paar Röhren. Daten fließen, wie könnte es anders sein, durch unsere Applikation hindurch - Pure Functions entsprechen Eingang und Ausgang. Wenn wir diese Verkettung nicht irgendwo unterbrechen, werden wir nie eine Ausgabe erhalten, was unsere Software nutztlos macht.

Wir stellen Komposition als Design Prinzip über alles andere. Denn so bleibt unsere App einfach und verständlich. Kategorientheorie wird eine große Rolle bei der App Architektur spielen, wenn es darum get Seiteneffekte zu modellieren und die Korrektheit sicher zu stellen.

Wir sind an einem Punkt angelangt, wo es uns gut täte ein wenig von alle dem in der Praxis zu sehen. Lass uns also eine Beispiel Applikation machen.

[Kapitel 6: Beispiel Appllikation](ch6-de.md)

## Exercises

```js
var _ = require('ramda');
var accounting = require('accounting');

// Beispieldaten
var CARS = [
    {name: "Ferrari FF", horsepower: 660, dollar_value: 700000, in_stock: true},
    {name: "Spyker C12 Zagato", horsepower: 650, dollar_value: 648000, in_stock: false},
    {name: "Jaguar XKR-S", horsepower: 550, dollar_value: 132000, in_stock: false},
    {name: "Audi R8", horsepower: 525, dollar_value: 114200, in_stock: false},
    {name: "Aston Martin One-77", horsepower: 750, dollar_value: 1850000, in_stock: true},
    {name: "Pagani Huayra", horsepower: 700, dollar_value: 1300000, in_stock: false}
  ];

// Übung 1:
// ============
// Schreibe die unten sthende Funktion mit Hilfe von _.compose() um. Hinweis: _.prop() ist curried.
var isLastInStock = function(cars) {
  var last_car = _.last(cars);
  return _.prop('in_stock', last_car);
};

// Übung 2:
// ============
// nutze _.compose(), _.prop() und _.head() um den Namen des ersten Autos zu ermitteln.
var nameOfFirstCar = undefined;


// Übung 3:
// ============
// Nutze die Hilfsfunktion _average um averageDollarValue durch Komposition zu refactoren
var _average = function(xs) { return _.reduce(_.add, 0, xs) / xs.length; }; // <- Nicht anfassen

var averageDollarValue = function(cars) {
  var dollar_values = _.map(function(c) { return c.dollar_value; }, cars);
  return _average(dollar_values);
};


// Übung 4:
// ============
// Schreibe mit compose eine Funktion: sanatizeNames() die eine Liste der Auto Bezeichnungen in Kleinbuchstaben und mit Unterstrichen statt Leerzeichen zurück gibt: z:B. sanatizeNamessanitizeNames([{name: "Ferrari FF", horsepower: 660, dollar_value: 700000, in_stock: true}]) //=> ["ferrari_ff"].

var _underscore = _.replace(/\W+/g, '_'); //<-- leave this alone and use to sanitize

var sanitizeNames = undefined;


// Bonus 1:
// ============
// Refactor availablePrices mit compose.

var availablePrices = function(cars) {
  var available_cars = _.filter(_.prop('in_stock'), cars);
  return available_cars.map(function(x){
    return accounting.formatMoney(x.dollar_value);
  }).join(', ');
};


// Bonus 2:
// ============
// Mach die Funktion Pointfree. Hinweis: Du kannst _.flip() benutzen.

var fastestCar = function(cars) {
  var sorted = _.sortBy(function(car){ return car.horsepower }, cars);
  var fastest = _.last(sorted);
  return fastest.name + ' is the fastest';
};
```

[lodash-website]: https://lodash.com/
[underscore-website]: http://underscorejs.org/
[ramda-website]: http://ramdajs.com/
[refactoring-book]: http://martinfowler.com/books/refactoring.html
[extract-method-refactor]: http://refactoring.com/catalog/extractMethod.html
