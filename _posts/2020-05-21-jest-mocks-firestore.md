---
layout: post
title: "Mockowanie Firestore"
date: 2020-05-21 07:30:00 +0200
tags: [javascript, node, firebase, firestore]
---

Obecnie pracuję przy projekcie, w którym używamy Firebase. Do _cloud functions_ używamy NodeJS, jako bazy danych Firestore, a do testów używany jest Jest. Pojawił się problem, jak zamockować Firestore w testach jednostkowych.

Firebase poleca używania do testów [emulatora](https://firebase.google.com/docs/rules/unit-tests), ale to rozwiązanie bardziej do testów integracyjnych, niż do testów jednostkowych i nie byliśmy z niego zbyt zadowoleni.

# Własne mocki

Na szczęście ostatecznie Firestore to moduł, który możemy zamockować. Chociaż nie jest to trywialne.

Załóżmy, że mamy taką funkcję:

```javascript
const firebaseAdmin = require("firebase-admin");

async function createPost(postId, content) {
  const document = { ...content };

  // ...

  return firebaseAdmin
    .firestore()
    .collection("Posts")
    .doc(postId)
    .create(document);
}
```

przyjmujemy id dokumentu i jego zawartość, przetwarzamy to jakoś i wsadzamy do Firestore.

Spróbujmy to zamockować.

```javascript
// test file
const { when } = require("jest-when");

const collectionMock = jest.fn();
const documentsMock = jest.fn();

const getMock = jest.fn();
const updateMock = jest.fn();
const createMock = jest.fn();

const documentMock = {
  get: getMock,
  update: updateMock,
  set: createMock,
};

jest.spyOn(admin, "firestore", "get").mockImplementation(() =>
  jest.fn(() => {
    collection: collectionMock;
  })
);

when(collectionMock).calledWith("Posts").mockReturnValue({
  doc: documentsMock,
});

describe("Mocking Firestore", () => {
  it("should create", () => {
    const postId = "123";
    const content = { header: "Hello world" };

    when(documentsMock).calledWith(postId).mockReturnValue(documentMock);

    // ... - reszta testu i asercje na createMock
  });
});
```

Kod jest niekompletny, bo dodatkowo po drodze trzeba zainicjalizować `firebase-admin`, wyczyścić mocki itp., ale chyba oddaje ogólną ideę. Tworzymy sobie mocki, podmieniamy wszystko co po drodze jest używane w kodzie produkcyjnym, uruchamiamy test i możemy robić asercje na naszych mockach.

Sporo roboty i nie wygląda to za pięknie, ale zadziałało. Można to wyciągnąć do osobnego modułu, wyeksportować funkcję do mockowania i konkretne mocki - wtedy będzie używalne w wielu miejscach i testy będą ładniej wyglądały. Ale może ktoś już coś takiego zrobił?

# Paczka firestore-jest-mock

Chwila w Googlu i znalazłem paczkę [firestore-jest-mock](https://www.npmjs.com/package/firestore-jest-mock), która wygląda bardzo fajnie i robi w zasadzie to, co my wcześniej, tylko lepiej i w większym zakresie.

Początkowo odrzuciliśmy ją, bo niestety zakładali, że projekt, który jej używa ma zależności zarówno do `firebase`, jak i do `firebase-admin`, podczas gdy my w projekcie używamy tylko `firebase-admin`. Na szczęście udało mi się to [poprawić](https://github.com/Upstatement/firestore-jest-mock/pull/28). Na ten moment (2020-05-21) poprawka jest tylko w branchu `master`, więc paczkę trzeba ściągnąć z gita, a nie z npm.

To już wygląda dużo ładniej:

```javascript
const { mockFirebase } = require("firestore-jest-mock");
const { mockSet } = require("firestore-jest-mock/mocks/firestore");

mockFirebase({
  database: {
    Posts: [],
  },
});

// uwaga: musi być po `mockFirebase`
require("firebase-admin").initializeApp();

// ... testy
```

Importujemy sobie funkcję `mockFirebase` i odpowiednie mocki, wołamy `mockFirebase` z zawartością bazy, jaką chcemy (pod kluczem `database`), inicjalizujemy aplikację i lecimy z testami. Tutaj uwaga na inicjalizację - musi być po `mockFirebase` - inaczej Firebase nie zaczyta mocków, tylko domyślną implementację.

Do wybranej kolekcji (tutaj `Posts`) można dodać sobie dokumenty i będą działały przy pobraniu ich przez `get`. Trzeba jednak pamiętać, że to nie jest emulator Firestore w pamięci, a jedynie mocki. Więc jeśli stworzymy dokument przy użyciu `set`, to nie będzie on dostępny przy `get`, a możemy jedynie sprawdzić na `mockSet` co się zadziało.

Paczka udostępnia sporo mocków, nie tylko te trzy, które implementowaliśmy na począku. Dodatkowo potrafi też emulować moduł `auth` w Firebase, ale tej funkcjonalności nie używaliśmy. Po więcej szczegółów odsyłam do [dokumentacji](https://www.npmjs.com/package/firestore-jest-mock).
