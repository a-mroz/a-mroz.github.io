---
layout: post
title: "Hoisting w javascripcie"
date: 2020-05-18 06:00:00 +0200
tags: [javascript, node]
---

W nawiązaniu do (przedostatniego posta)[_posts/2020-05-15-javascript-hoisting-jest.md] kilka słów o hostingu w javascripcie.

# Co to takiego?

W uproszczeniu kompilator/interpreter javascript przenosi sobie (wirtualnie) deklaracje (bez definicji!) początek swojego _scope_. Jeśli coś jest zdefiniowane w ramach funkcji, to na początek funkcji, a jeśli jest zdefiniowane poza funkcją, to na początek _global-scope_.

Jest to pokłosie tego, że kod javascript (przynajmniej oryginalnie) podczas wykonania był przechodzony przez interpreter dwukrotnie. Pierwsza faza przetwarzała deklaracje zmiennych i funkcji, a kod był wykonywany dopiero podczas drugiej fazy. Takie coś pozwala, przykładowo, na odwołanie się do funkcji, która jest zadeklarowana poniżej.

# Deklaracja, a definicja.

```javascript
let name; // deklaracja
name = "Adam"; // definicja
```

# Przykłady

Zobaczmy hoisting w praktyce:

#### Funkcje

```javascript
fn();

function fn() {
  console.log("Hello from fn()");
}
```

tutaj wszystko zadziała - dostaniemy `Hello from fn()`. Wirtualnie, podczas pierwszego przejścia, javascript zrobi sobie coś typu:

```javascript
function fn;

fn();

function fn() {
 console.log('Hello from fn()');
}
```

czyli przeniesie sobie _deklarację_ funkcji `fn` do góry i przy `fn()` zobaczy, że funkcja fn będzie gdzieś zadeklarowana, więc pozwoli na jej wywołanie (po drodze dobierając się do jej _definicji_).

No dobrze, to jest użyteczne, chociaż zdaje się, że do osiągnięca także innymi zabiegami. Zobaczmy, kiedy to nie zadziała i co może pójść nie tak.

Funkcje zdefiniowane w taki sposób będą hoistowane. Ale kiedy zdefiniujemy je jako wyrażenie (_expression_), to już nie:

```javascript
fn();

const fn = function () {
  console.log("Hello from fn()");
};
```

tutaj dostaniemy `ReferenceError: Cannot access 'fn' before initialization` - deklaracja `fn` nie zostanie zostanie przeniesiona na górę i

#### Zmienne

Zobaczmy teraz czemu `const` i `let` wprowadzone w ES6 sprawiły, że javascript stał się (trochę bardziej ;) ) użytecznym językiem.

Co wypisze taki kod?

```javascript
console.log(`Hello ${name}`);

var name = "Adam";
```

Osobiście spodziewałbym się, że dostaniemy błąd referencji. A jednak: `Hello undefined`. Czemu? Tak jak przy funkcjach, javascript przeniesie sobie deklarację zmiennej na początek _scope_, a definicję zostawi niżej:

```javascript
var name; // undefined
console.log(`Hello ${name}`);

name = "Adam";
```

Podczas wykonania, kiedy robimy `console.log`, to zmienna ma wartość `udefined`, bo jeszcze nie doszliśmy do jej definicji.

Coś, co przy funkcjach było użyteczne, tutaj powoduje raczej WTF w głowach programistów (i prawdopodobnie mnóstwo błędów),

# TODO

Use strict
let i const
classes
podsumowanie

Kod przykładów można znaleźć [tutaj](), a po więcej szczegółów zachęcam do zerknięcia na [ten tutorial](https://scotch.io/tutorials/understanding-hoisting-in-javascript).
