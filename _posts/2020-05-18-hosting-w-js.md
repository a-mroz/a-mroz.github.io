---
layout: post
title: "Hoisting w javascripcie"
date: 2020-05-18 20:46:00 +0200
tags: [javascript, node]
---

W nawiązaniu do [przedostatniego posta]({% post_url 2020-05-15-javascript-hoisting-jest %}) kilka słów o hostingu w javascripcie.

# Co to ten hoisting?

W dużym uproszczeniu można sobie wyobrazić, że kompilator/interpreter javascript przenosi sobie (wirtualnie, kod nie jest zmieniany) **deklaracje** na początek swojego _scope_. Jeśli coś jest zdefiniowane w ramach funkcji, to na początek funkcji, a jeśli jest zdefiniowane poza funkcją, to na początek _global-scope_.

Natomiast w trochę mniejszym uproszczeniu, javascript używa czegoś co nazywa się _lexical scope_. Jest to coś w rodzaju słownika/mapy, gdzie trzyma sobie zmienne itp. widoczne w obecnym _scope_.

Kod javascript podczas wykonania jest (a przynajmniej oryginalnie był) czytany przez interpreter dwukrotnie.
Pierwsza faza przetwarza **deklaracje** zmiennych i funkcji przenosząc je do _lexical scope_, a kod jest wykonywany dopiero podczas drugiej fazy. Takie coś pozwala, przykładowo, na odwołanie się do funkcji, która jest zadeklarowana poniżej wywołania.

# Deklaracja a definicja

Czemu podkreślam, że podczas pierwszego przejścia przetwarzane są tylko deklaracje? Co to w ogóle za różnica deklaracja a definicja? Spójrzmy na to:

```javascript
var name; // deklaracja
name = "Adam"; // definicja
```

W tym przykładzie `var name` to deklaracja, a dopiero `name = "Adam"` to definicja. Po pierwszym przejściu, dla interpretera, będzie to wyglądało mniej więcej tak:

```javascript
lexicalScope = {
  name: undefined,
};

lexicalScope.name = "Adam";
```

nawet jeśli zrobilibyśmy to w jednej linijce (`var name = "Adam"`).

Zmienne zadeklarowane przy użyciu `var` są inicjalizowane na `undefined`, więc jeśli spróbowalibyśmy wykorzystać tą zmienną przed jej definicją, to miałaby wartość `undefined`.

# Przykłady

Spójrzmy na kilka przykładów.

### Funkcje

```javascript
fn();

function fn() {
  console.log("Hello from fn()");
}
```

tutaj wszystko zadziała - dostaniemy `Hello from fn()`. Wirtualnie, podczas pierwszego przejścia, javascript znajdzie sobie deklarację `fn`, wpisze ją do swojego _lexical scope_ i podczas drugiego prześcia będzie ją widział, chociaż w pliku jest zadeklarowana niżej. Podczas drugiego przejścia mamy już coś typu:

```javascript
lexicalScope = {
  fn: < function >
}

lexicalScope.fn(); // lookup w lexicalScope

function fn() {
 console.log("Hello from fn()");
}
```

No dobrze, to jest użyteczne. Funkcje zdefiniowane w taki sposób będą hoistowane. Ale zobaczmy,coś może pójść nie tak.

Otóż kiedy zdefiniujemy je jako wyrażenie (_expression_), to już nie zadziała:

```javascript
fn();

fn = function () {
  console.log("Hello from fn()");
};
```

tutaj dostaniemy `ReferenceError: Cannot access 'fn' before initialization`.

A to dlatego, że kiedy interpreter napotka kod taki jak w pierwszym przypadku, to potraktuje całość jako deklarację funkcji - nie rozdzieli tego na deklarację i definicję. Drugi przypadek potraktuje jako deklarację zmiennej i tutaj już do definicję zobaczy dopiero przy drugim przejściu - deklaracja `fn` zostanie zostanie przeniesiona do _lexical scope_, ale definicja już nie, więc podczas wykonania będzie miała wartość `undefined`.

### Zmienne

Co wypisze taki kod (wskazówka była wyżej)?

```javascript
console.log(`Hello ${name}`);

var name = "Adam";
```

Można spodziewać się, że dostaniemy błąd referencji. A jednak: `Hello undefined`. Czemu? Tak jak przy funkcji jako wyrażeniu (bo to _de facto_ ten sam scenariusz), javascript przeniesie sobie deklarację zmiennej na początek _scope_, a definicję zostawi niżej:

```javascript
lexicalScope = {
  name: undefined,
};

console.log(`Hello ${name}`);

name = "Adam";
```

Podczas wykonania, kiedy robimy `console.log`, to zmienna ma wartość `undefined`, bo jeszcze nie doszliśmy do jej definicji. Tutaj nie dostaniemy wyjątku, bo `undefined` to normalna wartość, której używamy, a nie próbujemy traktować jej jako funkcji i wywoływać jej.

#### let i const

Ponieważ `var` powoduje spore WTF u programistów (i dużo błędów), to ES6 wprowadziło deklaracje zmiennych przy pomocy`let` i `const`. Zobaczmy teraz czemu to sprawiło, że javascript stał się (trochę bardziej ;) ) użytecznym językiem.

```javascript
console.log(`Hello ${name}`);
let name = "Adam"; // to samo dla `const`
```

dostaniemy `ReferenceError: Cannot access 'name' before initialization`. A to dlatego, że zmienne `let` i `const` także są hoistowane, ale nie są inicjalizowane na `undefined`, tak jak zmienne typu `var`. Podczas pierwszego przejścia zmienna zostanie wpisana do _lexical scope_ (stąd `before initialization`, a nie `is not defined`), ale inicjalizacja nastąpi dopiero po napotkaniu jej definicji. W tym przypadku definicja następuje po próbie jej wykorzystania, więc dostaniemy wyjątek.

### Klasy

Z klasami będzie jak z `let` i `const`. Deklaracja klasy będzie hoistowana, ale nie będzie zaincjalizowana. To oznacza, że klasy możemy użyć dopiero po jej definicji:

```javascript
const cat = new Cat("Filemon");

class Cat {
  constructor(name) {
    this.name = name;
  }
}
```

nie zadziała: `ReferenceError: Cannot access 'Cat' before initialization`.

Jeśli spróbujemy zrobić to samo, ale przez zadeklarowanie klasy jako wyrażenia:

```javascript
const filemon = new Cat("Filemon");

// albo `const/let Cat`
var Cat = class {
  constructor(name) {
    this.name = name;
  }
};
```

to przy wykorzystaniu `var` dostaniemy `TypeError: Cat is not a constructor` (prawda, w końcu to `undefined`), a przy `const`/`let` spodziewany `ReferenceError`.

# Podsumowanie

Hoisting w javascripcie bywa mylący, ale mam nadzieję, że ten post komuś trochę rozjaśnił temat. Mi napisanie tego na pewno pomogło w zrozumieniu mechanizmu hoistingu.

Ze swojej strony mocno zachęcam do zrobienia sobie i innym przysługi i używania `let` i `const` zamiast `var` wszędzie, gdzie się da.

Kod przykładów można znaleźć [tutaj](https://github.com/a-mroz/hoisting-examples), a po więcej szczegółów zachęcam do zerknięcia na [ten tutorial](https://scotch.io/tutorials/understanding-hoisting-in-javascript) i
[ten wpis](https://blog.bitsrc.io/hoisting-in-modern-javascript-let-const-and-var-b290405adfda).
