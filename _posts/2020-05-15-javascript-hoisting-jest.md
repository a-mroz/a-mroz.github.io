---
layout: post
title: "'out-of-scope variables' w jest a hoisting"
date: 2020-05-15 21:07:00 +0200
tags: [javascript, node, jest]
---

Dzisiejszy, ciekawy problem z mockowaniem przy pomocy biblioteki `jest`.

Mamy (w NodeJS) test typu:

```javascript
function fn() {
  console.log("Hello from mock!");
}

jest.mock("some-module", () => fn);

describe("Test", () => {
  it("works", () => {
    //...
    expect(true).toBe(true);
  });
});
```

Na pierwszy rzut oka wszystko wygląda ok. Ale przy uruchomieniu dostajemy błąd:

```
The module factory of `jest.mock()` is not allowed to reference any out-of-scope variables.
Invalid variable access: fn
Whitelisted objects: Array, ArrayBuffer, Atomics, BigInt, BigInt64Array, BigUint64Array, Boolean, Buffer, DataView, Date, Error, EvalError, Float32Array, Float64Array, Function, GLOBAL, Generator, GeneratorF
unction, Infinity, Int16Array, Int32Array, Int8Array, InternalError, Intl, JSON, Map, Math, NaN, Number, Object, Promise, Proxy, RangeError, ReferenceError, Reflect, RegExp, Set, SharedArrayBuffer, String, Symbo
l, SyntaxError, TextDecoder, TextEncoder, TypeError, URIError, URL, URLSearchParams, Uint16Array, Uint32Array, Uint8Array, Uint8ClampedArray, WeakMap, WeakSet, WebAssembly, arguments, clearImmediate, clearInterv
al, clearTimeout, console, decodeURI, decodeURIComponent, encodeURI, encodeURIComponent, escape, eval, expect, global, globalThis, isFinite, isNaN, jest, parseFloat, parseInt, process, queueMicrotask, require, r
oot, setImmediate, setInterval, setTimeout, undefined, unescape.
Note: This is a precaution to guard against uninitialized mock variables. If it is ensured that the mock is required lazily, variable names prefixed with `mock` (case insensitive) are permitted.
```

To co się tu dzieje spowodowane jest [hoistingiem]({% post_url 2020-05-18-hosting-w-js %}). Niby człowiek wie, czytał, ale i tak może go to kopnąć w tyłek.

`jest.mock` zostanie przez `babel-plugin-jest-hoist` wyniesiony na górę pliku. I co prawda zobaczy deklarację funkcji `fn`, bo deklaracje funkcji są hoistowane. Ale nie zobaczy jej definicji i będzie marudził, [by design](https://github.com/facebook/jest/issues/9730).

# Rozwiązanie

Rekomendowanym rozwiązaniem (patrz ticket podlinkowany wyżej) byłoby przeniesienie funkcji, do innego pliku i zaimportowanie jej tam, gdzie chcemy użyć jej do mockowania. To powinno zadziałać, ale w tym przypadku miałem ograniczone pole manewru i niespecjalnie mogłem to zrobić.

Drugą opcją, tak jak mówi notatka w błędzie, jeśli jesteśmy pewni, że zmienna będzie zaincjalizowana, to możemy zmienić nazwę funkcji tak, żeby zaczynała się na `mock`. Wtedy `jest` nam to przepuści:

```javascript
function mockFn() {
  console.log("Hello from mock!");
}

jest.mock("some-module", () => mockFn);

describe("Test", () => {
  it("works", () => {
    expect(true).toBe(true);
  });
});
```

Jednak ciekawszym rozwiązaniem jest użycie `jest.doMock` zamiast `jest.mock`. Różnią się tym, że `doMock` nie jest hoistowany.

```javascript
function fn() {
  console.log("Hello from mock!");
}

jest.doMock("some-module", () => fn);

describe("Test", () => {
  it("works", () => {
    expect(true).toBe(true);
  });
});
```

Taki mock nie będzie wyniesiony na początek pliku, więc będzie widział zainicjalizowane `fn`.

Poza sytuacjami typu tej, która była opisana powyżej,jest to też przydatne, kiedy chcemy zamockować moduł kilkukrotnie w ramach jednego pliku - na przykład żeby mieć mockowanie per test, a nie jedno globalne.

Po przykład i więcej informacji zapraszam [do dokumentacji](https://jestjs.io/docs/en/jest-object#jestdomockmodulename-factory-options).
