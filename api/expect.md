# expect

Die `expect`-Funktion in Vitest wird verwendet, um Assertions in Tests durchzuführen.

Offizielle Dokumentation: [https://vitest.dev/api/expect.html](https://vitest.dev/api/expect.html)

## objectContaining

Prüft, ob ein Objekt bestimmte Eigenschaften enthält.

Offizielle Dokumentation: [https://vitest.dev/api/expect.html#expect-objectcontaining](https://vitest.dev/api/expect.html#expect-objectcontaining)

```javascript
import { expect, test } from 'vitest'

// Beispiel #1
expect(chatCompletion).toEqual(expect.objectContaining(expectedResponse))

// Beispiel #2
test('basket has empire apples', () => {
  const basket = {
    varieties: [
      {
        name: 'Empire',
        count: 1,
      }
    ],
  }
  expect(basket).toEqual({
    varieties: [
      expect.objectContaining({ name: 'Empire' }),
    ]
  })
})
```

## toThrowError()

Prüft, ob eine Funktion einen Fehler wirft.

Offizielle Dokumentation: [https://vitest.dev/api/expect.html#tothrowerror](https://vitest.dev/api/expect.html#tothrowerror)

### Synchrone Funktionen

```javascript
import { expect, test } from 'vitest'

function getFruitStock(type: string) {
  if (type === 'pineapples') {
    throw new Error('Pineapples are not in stock')
  }

  // Do some other stuff
}

test('throws on pineapples', () => {
  // Test that the error message says "stock" somewhere: these are equivalent
  expect(() => getFruitStock('pineapples')).toThrowError(/stock/)
  expect(() => getFruitStock('pineapples')).toThrowError('stock')

  // Test the exact error message
  expect(() => getFruitStock('pineapples')).toThrowError(
    /^Pineapples are not in stock$/,
  )
})
```

### Asynchrone Funktionen

```javascript
function getAsyncFruitStock() {
  return Promise.reject(new Error('empty'))
}

test('throws on pineapples', async () => {
  await expect(() => getAsyncFruitStock()).rejects.toThrowError('empty')
})
``` 