---
layout: post
title: "Hoisting w javascripcie"
date: 2020-05-18 06:00:00 +0200
tags: [javascript, node]
---

W nawiązaniu do (przedostatniego posta)[_posts/2020-05-15-javascript-hoisting-jest.md] kilka słów o hostingu w javascripcie.

# Co to ten hoisting?

W uproszczeniu kompilator/interpreter javascript przenosi sobie (wirtualnie, kod nie jest zmieniany) deklaracje (bez definicji!) początek swojego _scope_. Jeśli coś jest zdefiniowane w ramach funkcji, to na początek funkcji, a jeśli jest zdefiniowane poza funkcją, to na początek _global-scope_.

Jest to pokłosie tego, że kod javascript (przynajmniej oryginalnie) podczas wykonania był przechodzony przez interpreter dwukrotnie. Pierwsza faza przetwarzała deklaracje zmiennych i funkcji przenosząc je do _lexical scope_, a kod był wykonywany dopiero podczas drugiej fazy. Takie coś pozwala, przykładowo, na odwołanie się do funkcji, która jest zadeklarowana poniżej.

# Deklaracja, definicja i _lexical scope_

```javascript
let name; // deklaracja
name = "Adam"; // definicja
```

deklaracje są hoistowane a definicje nie.

# Przykłady

Zobaczmy hoisting w praktyce:

#### Funkcje

```javascript
fn();

function fn() {
  console.log("Hello from fn()");
}
```

tutaj wszystko zadziała - dostaniemy `Hello from fn()`. Wirtualnie, podczas pierwszego przejścia, javascript znajdzie sobie deklarację `fn`, wpisze ją do swojego _lexical scope_ i podczas drugiego prześcia będzie ją widział, chociaż w pliku jest zadeklarowana niżej. Można sobie wyobrazić, że podczas drugiego przejścia mamy coś typu:

```javascript
lexicalScope = {
  fn: < function >
}

lexicalScope.fn(); // lookup do lexicalScope

function fn() {
 console.log('Hello from fn()');
}
```

No dobrze, to jest użyteczne. Zobaczmy, kiedy to nie zadziała i co może pójść nie tak.

Funkcje zdefiniowane w taki sposób będą hoistowane. Ale kiedy zdefiniujemy je jako wyrażenie (_expression_), to już nie:

```javascript
fn();

fn = function () {
  console.log("Hello from fn()");
};
```

tutaj dostaniemy `ReferenceError: Cannot access 'fn' before initialization` - deklaracja `fn` zostanie zostanie przeniesiona do _lexical scope_, ale definicja już nie, więc podczas wykonania będzie miała wartość `undefined`.

#### Zmienne

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

Podczas wykonania, kiedy robimy `console.log`, to zmienna ma wartość `undefined`, bo jeszcze nie doszliśmy do jej definicji.

Coś, co przy funkcjach było użyteczne, tutaj powoduje raczej WTF w głowach programistów (i prawdopodobnie mnóstwo błędów).

Dlatego ES6 wprowadziło deklaracje zmiennch przy pomocy`let` i `const`. Zobaczmy teraz czemu to sprawiło, że javascript stał się (trochę bardziej ;) ) użytecznym językiem. Zobaczmy:

```javascript
console.log(`Hello ${name}`);
let name = "Adam"; // to samo dla `const`
```

dostaniemy `ReferenceError: Cannot access 'name' before initialization`. A to dlatego, że zmienne `let` i `const` także są hoistowane, ale nie są inicjalizowane na `undefined`, tak jak zmienne typu var. Podczas pierwszego przejścia zmienna zostanie wpisana do _lexical scope_ (stąd `before initialization`, a nie `is not defined`), ale inicjalizacja nastąpi dopiero po napotkaniu jej definicji. W tym przypadku definicja następuje po próbie jej wykorzystania, więc dostaniemy wyjątek.

# Klasy

Z klasami będzie jak z `let` i `const`. Deklaracja klasy będzie hoistowana, ale nie będzie zaincjalizowana.
#TODO

# Podsumowanie

Kod przykładów można znaleźć [tutaj](), a po więcej szczegółów zachęcam do zerknięcia na [ten tutorial](https://scotch.io/tutorials/understanding-hoisting-in-javascript).
https://blog.bitsrc.io/hoisting-in-modern-javascript-let-const-and-var-b290405adfda
