
Obecnie pracuję przy projekcie, w którym używamy NodeJS (funkcje), Firebase z Firestore, do testów używany jest Jest. Pojawił się problem, jak zamockować Firestore w testach jednostkowych.

Firebase poleca używania do testów emulatora https://firebase.google.com/docs/rules/unit-tests Ale to rozwiązanie bardziej do testów integracyjnych, niż do testów jednostkowych.

Chwila w Googlu i znalazłem https://www.npmjs.com/package/firestore-jest-mock która wygląda bardzo fajnie. Niestety polegają tutaj na tym, że w projekcie używamy biblioteki `firebase`, podczas gdy mamy tylko `firebase-admin`.


Przykładowy kod:
```javascript
const firebaseAdmin = require('firebase-admin')

async function createPost(postId, content) {
    const document = { ...content }

    // ...

    return firebaseAdmin.firestore().collection('Users').doc(uid).create(document)
}

```

```javascript
const admin = require('firebase-admin')
const { when } = require('jest-when')

// admin.initializeApp() ?
// jest.spyOn(admin, 'initializeApp').mockImplementation(jest.fn) ?


describe('Mocking Firestore', () => {

    beforeAll(() => {
        const collectionMock = jest.fn()

        jest
            .spyOn(admin, 'firestore', 'get')
            .mockImplementation(() => jest.fn(() => {
                collection: collectionMock
            }))

        


    })

})
```