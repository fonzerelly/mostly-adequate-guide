# Kapitel 4: Currying

## Can't live if livin' is without you
Mein Vater hat mir einmal erklärt, dass es Dinge gibt ohne die man Leben kann bis man sie sich besorgt hat. Eine Mikrowelle wäre so ein Ding, Smart phones ein anderes. Die Älteren unter uns erinnern sich an ein erfülltes Leben ohne Internet. Für mich gehört currying auf diese Liste.

Das Konzept ist einfach: Man kann eine Funktion mit weniger Argumenten aufrufen als sie erwartet. Heraus kommt eine Funktion, welche die restlichen Argumente entgegen nimmt.

Man kann die nach belieben alle Argumente auf einmal übergeben oder einfach Häppchenweise.

```js
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var increment = add(1);
var addTen = add(10);

increment(2);
// 3

addTen(2);
// 12
```

Hier haben wir eine Funktion `add`, die ein Argument entgegen nimmt und eine Funktion zurückliefert. Wird diese Aufgerufen, erinnert sich die Funktion an das erste Argument ab da über die Closure. Solche Funktionen für jedes einzelne Argument zu schreiben ist ein wenig schmerzhaft. Deshalb können wir eine spezielle Hilfsfunktion namens `curry`verwenden, die das definieren und Aufrufen sollcher Funktionen vereinfacht.

Wir wollen mal eben ein paar curried Functions zum Spass aufsetzen.

```js
var curry = require('lodash.curry');

var match = curry(function(what, str) {
  return str.match(what);
});

var replace = curry(function(what, replacement, str) {
  return str.replace(what, replacement);
});

var filter = curry(function(f, ary) {
  return ary.filter(f);
});

var map = curry(function(f, ary) {
  return ary.map(f);
});
```
Ich folge hier einem einfach aber wichtigen Muster. Ich habe die Daten mit denen wir operieren werden (String, Array) strategisch als letztes Argument positioniert. Wenn wir sie benutzen wird es klarer.

```js
match(/\s+/g, "hello world");
// [ ' ' ]

match(/\s+/g)("hello world");
// [ ' ' ]

var hasSpaces = match(/\s+/g);
// function(x) { return x.match(/\s+/g) }

hasSpaces("hello world");
// [ ' ' ]

hasSpaces("spaceless");
// null

filter(hasSpaces, ["tori_spelling", "tori amos"]);
// ["tori amos"]

var findSpaces = filter(hasSpaces);
// function(xs) { return xs.filter(function(x) { return x.match(/\s+/g) }) }

findSpaces(["tori_spelling", "tori amos"]);
// ["tori amos"]

var noVowels = replace(/[aeiou]/ig);
// function(replacement, x) { return x.replace(/[aeiou]/ig, replacement) }

var censored = noVowels("*");
// function(x) { return x.replace(/[aeiou]/ig, "*") }

censored("Chocolate Rain");
// 'Ch*c*l*t* R**n'
```

Was hier demonstriert wird, ist die Fähigkeit eine Funktion mit einem oder zwei Argumenten "vor zu laden" um eine neue Funktion zu erhalten, die sich an diese Argumente erinnert.

Ich möchte Dich ermutigen, `npm install lodash` auszuführen, den code oben zu kopieren und in der Repl damit herum zu experimentieren. Du kannst das auch im Browser ausprobieren, wo lodash oder ramda verfügbar ist.

## Mehr als ein Wortspiel / Spezialsoße
Currying ist für viele Dinge nützlich. Wir können eine neue Funktion bauen, nur indem wir unseren Basis Funktionen ein paar Argumente mitgegeben haben. So geschen in `hasSpaces`, `findSpaces` und `censored`.

Wir können auch jede funktion, die auf einem Element arbeitet in eine Funktion transformieren die auf Arrays arbeitet in dem wir sie einfach in ein `map` einbetten:

```js
var getChildren = function(x) {
  return x.childNodes;
};

var allTheChildren = map(getChildren);
```

Einer Funktion weinger Argumente zu geben, als sie erwartet nennt man üblicherweise *partial application* (teilweise Anwendung). Eine Funktion nur Teilweise anzuwenden kann sehr viel Boiler Plate Code einsparen. Betrachten wir z.B. wie die oben stehende Funktion `allTheChildren` mit einem nicht gecurrieten `map` von lodash (achte auf die andere Reihenfolge der Argumente):

```js
var allTheChildren = function(elements) {
  return _.map(elements, getChildren);
};
```

Typischerweise definieren wir keine Funktionen die auf Arrays arbeiten, denn wir können einfach `map(getChildren)` inline aufrufen. Das Gleiche gilt für `sort`, `filter` und andere Funktionen höherer Ordnung (Higher Order Function: Eine Funktion die eine andere Funktion entgegen nimmt oder zurückliefert.)

Als wir über *Pure Functions* gesprochen haben, haben wir gesagt, dass sie eine Eingabe zu einer Ausgabe machen. Currying macht genau das: Jedes einzelne Argument liefert eine neue Funktion, die die verbleibenden Argumente erwartet. Das Sportfreund ist eine Eingabe zu einer Ausgabe.

Auch wenn die Ausgabe eine andere Funktion ist - das qualifiziert curry als pure. Wir erlauben zwar mehr als ein Argument pro Aufruf, aber das ist eher Bequemlichkeit um die extra `()` zu vermeiden.


## Zusammenfassung

Currying ist sehr parktisch und ich genieße es täglich mit Curried Functions zu arbeiten. Es ist ein Werkzeug für den Gürtel, das Funktionale Programmierung weinger verbos und umständlich.

Wir können neue, nützliche Funktionen on the fly machen, einfach in dem wir ein paar Argumente übergeben. Und als Bonus bewahren wir die mathematische Definition einer Funktion obwohl wir mehr als ein Argument übergeben.

Wir besorgen uns ein weiteres essentielles Werkzeug namens `compose`

[Kapitel 5: Coden durch Komposition](ch5-de.md)

## Aufgaben

Eine kleine Anmerkung bevor wir angangen. Wir werden eine library namens *ramda* benutzen, die jede Funktion per default curriet. Alternative könntest Du auch *lodash-fp* wählen, was das Gleiche tut und vom Autor von lodash geschrieben wurde. Beide funktionieren wunderbar und die Auswahl ist eher Geschmackssache.

[ramda](http://ramdajs.com)
[lodash-fp](https://github.com/lodash/lodash-fp)

Es gibt [Unit Tests](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises) gegen die Du Deine Übungen laufen lassen kannst während du sie programmierst, oder wenn Du magst, kannst Du sie bei die ersten Übungen auch einfach in eine javascript REPL kopieren.

Lösungsvorschläge findest Du im Code des [Repository dieses Buchs](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises/answers).
Answers are provided with the code in the [repository for this book](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises/answers)

```js
var _ = require('ramda');


// Aufgabe 1
//==============
// Schreibe die Funktion mit Hilfe von Partial Application so um, dass alle Argumente gelöscht werden können.

var words = function(str) {
  return _.split(' ', str);
};

// Aufgabe 1a
//==============
// Nutze map um eine neue Funktion zu schreiben, die mit Hilfe von words auf einem Array von Strings arbeiten kann.

var sentences = undefined;


// Aufgabe 2
//==============
// Schreibe die Funktion mit Hilfe von Partial Application so um, dass alle Argumente gelöscht werden können.

var filterQs = function(xs) {
  return _.filter(function(x){ return match(/q/i, x);  }, xs);
};


// Aufgabe 3
//==============
// Nutze die hilfs Funktion _keepHighest um max so zu vereinfachen, dass es
// keine Argumente mehr benötigt

// NICHT ANFASSEN:
var _keepHighest = function(x,y){ return x >= y ? x : y; };

//  BEARBEITE DIESE:
var max = function(xs) {
  return _.reduce(function(acc, x){
    return _keepHighest(acc, x);
  }, -Infinity, xs);
};


// Bonus 1:
// ============
// Bette Array's slice so ein, dass es Funktional und gecurried wird.
// //[1,2,3].slice(0, 2)
var slice = undefined;


// Bonus 2:
// ============
// Nutze slice um eine Funktion "take" zu definieren, die die ersten n Elemente des Strings zurück liefert. Mach Sie curried.
// Ergebnis für "Something" mit n=4 soll "Some" sein
var take = undefined;
```
