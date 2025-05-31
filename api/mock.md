# Mocking

## Modules

### External NPM Packages

#### Auto Mock

<details><summary>Click to expand..</summary>

### ❌ Problem: `importMock()` Inside `vi.mock()` = 💥 Infinite Recursion

Calling `importMock()` **inside** a `vi.mock()` block is a trap:
It tries to load the very module you're currently mocking → triggers `vi.mock()` again → **infinite loop** → boom.

---

### ✅ Correct Approach: Use `importOriginal`

Never use `importMock()` inside `vi.mock()`.
Instead, Vitest provides `importOriginal` exactly for this purpose:

```ts
vi.mock('some-module', async (importOriginal) => {
  const original = await importOriginal<typeof import('some-module')>()
  const { mockObject } = await import('vitest/mocker')
  return mockObject({ type: 'automock', spyOn: vi.spyOn }, original)
})
```

---

### ✅ Clean Solution: Mock Factory Pattern

Here’s a robust and reusable pattern using a hoisted mock factory. Example: mocking the `@pinecone-database/pinecone` module.

```ts
import { describe, it, expect, vi, beforeEach, type MockedObject } from 'vitest'
import { mockObject } from 'vitest/mocker'

type PineconeModule = typeof import('@pinecone-database/pinecone')
type MockedPineconeModule = MockedObject<PineconeModule>

const mockFactory = vi.hoisted(() => {
  let mockedModule: MockedPineconeModule

  const createAndStoreMockedModule = async (): Promise<MockedPineconeModule> => {
    const original = await vi.importActual<PineconeModule>('@pinecone-database/pinecone')
    const module = mockObject(
      {
        type: 'automock',
        spyOn: vi.spyOn,
        globalConstructors: { Object, Function, RegExp, Array, Map }
      },
      original
    ) as MockedPineconeModule

    mockedModule = module
    return module
  }

  return {
    getMockedPineconeModule: (): MockedPineconeModule => mockedModule,
    createAndStoreMockedModule
  }
})

vi.mock('@pinecone-database/pinecone', async () => {
  return mockFactory.createAndStoreMockedModule()
})
```

And then in your tests:

```ts
describe('PineconeService', () => {
  let service: PineconeService
  let mockedPinecone: MockedPineconeModule

  beforeEach(() => {
    mockedPinecone = mockFactory.getMockedPineconeModule()
    service = createStandardPineconeService()
  })

  it('✅ should initialize with correct API key and namespace', () => {
    expect(mockedPinecone.Pinecone).toHaveBeenCalledWith({ apiKey: env.PINECONE_API_KEY })
    expect(Reflect.get(service, '_namespace')).toBe(env.PINECONE_RULES_NAMESPACE)
  })
})
```

---

### 🧠 Recap

| Action                                | Context             | Status           |
| ------------------------------------- | ------------------- | ---------------- |
| ❌ `importMock()`                      | Inside `vi.mock()`  | ❌ Never          |
| ✅ `importOriginal()` + `mockObject()` | Inside `vi.mock()`  | ✅ Correct        |
| ✅ `importMock()`                      | Outside `vi.mock()` | ✅ Safe           |
| ✅ Use mock factory wrapper            | Anywhere            | 💪 Best practice |

</details>












<br><br>
________
________
<br><br>








# Class

## Method
```typescript
describe('Patients', () => {
    let patientServiceSpy: MockInstance<PatientService['getPatients']>

    beforeEach(() => {
        patientServiceSpy = vi.spyOn(PatientService.prototype, 'getPatients')
    })

    it.only('should return a filtered list of patients when patientId is provided', async() => {
        // Assuming '1' is a patientId that might exist or return an empty list,
        // which is fine for testing the filter mechanism.
        const patientIdToFilter = '1' 
        const response = await testServer.apiClient.get<IPatientsResponse>(
            `${API_PATH}/patients?patientId=${patientIdToFilter}`
        )

        expect(patientServiceSpy).toHaveBeenCalledWith(false, { patientId: patientIdToFilter })
        
        expect(response.status).toBe(200)
        expect(response.data).toHaveProperty('success', true)
        expect(response.data).toHaveProperty('patients')
        expect(Array.isArray(response.data.patients)).toBe(true)
        // Further assertions could be added here if we know specific data about patient '1'
        // For example, if patient '1' exists, we could check:
        // if (response.data.patients.length > 0) {
        //   expect(response.data.patients[0]).toHaveProperty('PATNR', patientIdToFilter);
        // }
    })
})
```











<br><br>
________
________
<br><br>



<details><summary>Click to expand..</summary>





### 🔍 Methoden zur direkten Prüfung (`.mock`-Objekt)

```ts
// Wurde der Spy überhaupt aufgerufen?
const wasCalled = patientServiceSpy.mock.calls.length > 0;

// Wie oft wurde er aufgerufen?
const callCount = patientServiceSpy.mock.calls.length;

// Mit welchen Argumenten beim 1. Call?
const firstCallArgs = patientServiceSpy.mock.calls[0];

// Mit welchen Argumenten beim letzten Call?
const lastCallArgs = patientServiceSpy.mock.calls.at(-1);

// Ergebnis des 1. Calls?
const firstCallResult = patientServiceSpy.mock.results[0]; // { type: 'return', value: ... }
```

---

### 💡 Beispiel

```ts
if (patientServiceSpy.mock.calls.length === 0) {
  console.warn('getPatients wurde nicht aufgerufen');
}

for (const call of patientServiceSpy.mock.calls) {
  console.log('Args:', call);
}

console.log('Return value beim ersten Aufruf:', patientServiceSpy.mock.results[0]?.value);
```

---

### 🧠 Pro-Tipp: Zugriff auf `this` und Instanzen

```ts
// Falls Methode auf einer Klasse mit `this` lief
patientServiceSpy.mock.instances // Alle `this`-Kontexte
```

---

### 🧬 Bonus: Use-Case ohne `expect`

```ts
if (callCount > 3) {
  doSomethingCool();
}
```
