---
layout: post
title: "NodeJS - jak sprawdzić czy moduł X jest dostępny"
date: 2020-05-16 11:15:00 +0200
tags: [javascript, node, jest, firebase]
---

Ostatnio potrzebowałem kawałka kodu do sprawdzenia, czy moduł o jakiejś nazwie jest dostępny w projekcie NodeJS.

Chodziło akurat o mockowanie modułów przy pomocy biblioteki `jest` w module pomocniczym, który możemy zaimportować do innego projektu, gdzie akurat może nie być części zależności.

W [tym przypadku](https://github.com/Upstatement/firestore-jest-mock/pull/28) biblioteka mockuje zależność `firebase` i `firebase-admin`, podczas gdy u nas w projekcie używamy tylko `firebase-admin`. A `jest` rzuca wyjątek, bardzo słusznie zresztą, jeśli każemy mu zamockować moduł, którego nie może znaleźć.

Najlepszym rozwiązaniem, jakie znalazłem, było użycie funkcji [`require.resolve`](https://nodejs.org/api/modules.html#modules_require_resolve_request_options).

Funkcja ta rozwiązuje ścieżkę do modułu, ale nie ładuje go niepotrzebnie, tylko zwraca do niego ścieżkę. Jeśli nie znajdzie modułu, to rzuca wyjątkiem. Można więc zrobić coś typu:

```javascript
try {
  require.resolve(moduleName);
  // do something
} catch (e) {
  // not found, console.info etc.
}
```
