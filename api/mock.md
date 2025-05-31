# Mocking

## Modules

### External NPM Packages

#### Auto Mock

<details><summary>Click to expand..</summary>


Option 1 - importActual():


<details><summary>Click to expand..</summary>

```

vi.mock('@pinecone-database/pinecone', async() => {
    const original = await vi.importActual<typeof import('@pinecone-database/pinecone')>('@pinecone-database/pinecone')
    const { mockObject } = await import('vitest/mocker')
    
    // Use mockObject to automatically mock the entire module
    const mocked = mockObject(
        {
            type: 'automock',
            spyOn: vi.spyOn,
            globalConstructors: {
                Object,
                Function,
                RegExp,
                Array,
                Map
            }
        },
        original
    )
    
    const mockedModule: IMockedPineconeModule = {
        ...mocked,
        Pinecone: mocked.Pinecone as ReturnType<typeof vi.fn>,
        Index: mocked.Index as ReturnType<typeof vi.fn>
    }
    
    // Store the mocked module in hoisted variable for test access
    hoistedMocks.setMockedPineconeModule(mockedModule)
    
    return mockedModule
})
```
- Full logic as in Option 3
  
</details>



<br><br>
<br><br>

Option 2 - importOriginal():


<details><summary>Click to expand..</summary>

```
// ==== Mocks ====
const hoistedMocks = vi.hoisted(() => {
    let mockedPineconeModule: IMockedPineconeModule | null = null
    
    return {
        getMockedPineconeModule: (): IMockedPineconeModule | null => mockedPineconeModule,
        setMockedPineconeModule: (module: ReadonlyDeep<IMockedPineconeModule>): void => {
            mockedPineconeModule = module as IMockedPineconeModule
        }
    }
})

vi.mock('@pinecone-database/pinecone', async importOriginal => {
    const original = await importOriginal<typeof import('@pinecone-database/pinecone')>()
    const { mockObject } = await import('vitest/mocker')
    
    // Use mockObject to automatically mock the entire module
    const mocked = mockObject(
        {
            type: 'automock',
            spyOn: vi.spyOn,
            globalConstructors: {
                Object,
                Function,
                RegExp,
                Array,
                Map
            }
        },
        original
    )
    
    const mockedModule: IMockedPineconeModule = {
        ...mocked,
        Pinecone: mocked.Pinecone as ReturnType<typeof vi.fn>,
        Index: mocked.Index as ReturnType<typeof vi.fn>
    }
    
    // Store the mocked module in hoisted variable for test access
    hoistedMocks.setMockedPineconeModule(mockedModule)
    
    return mockedModule
})

// ==== Tests ====
describe('PineconeService', () => {
    let service: PineconeService
    let mockedPinecone: IMockedPineconeModule

    const namespace = env.PINECONE_RULES_NAMESPACE
    const apiKey = env.PINECONE_API_KEY

    beforeEach(() => {
        // Get the mocked module
        const module = hoistedMocks.getMockedPineconeModule()
        if (!module) {
            throw new Error('Mocked Pinecone module not available')
        }
        mockedPinecone = module
        service = createStandardPineconeService()
    })

    describe('✅ Constructor', () => {
        it.only('sollte korrekt mit Standard-API-Key und Namespace initialisieren', () => {
            expect(mockedPinecone.Pinecone).toHaveBeenCalledWith({ apiKey })
            expect(Reflect.get(service, '_namespace')).toBe(namespace)
        })
    })
})
```

</details>




<br><br>
<br><br>

Not working - importMock()

<details><summary>Click to expand..</summary>

Exakt. Du hast den Haken erkannt, den viele übersehen:
👉 **`importMock()` innerhalb von `vi.mock()` für genau dasselbe Modul** ist ein verdammter 🌀 **Rekursions-Todeskreis**.

---

## 🔥 Warum passiert das?

```ts
vi.mock('@pinecone-database/pinecone', async () => {
  const { importMock } = await import('vitest/mocker')
  const mocked = await importMock('@pinecone-database/pinecone') // 💀
})
```

Sobald `importMock('@pinecone-database/pinecone')` aufgerufen wird,
versucht **Vitest**, das **gemockte** Modul zu importieren…
…aber **du bist ja gerade dabei**, es zu mocken.
👉 Zack. Infinite Loop. 🌀💥

---

## 🧠 Lösung: **`importOriginal` verwenden – nicht `importMock`**

Wenn du dich **innerhalb von `vi.mock()` befindest**, dann nimm immer:

```ts
vi.mock('@pinecone-database/pinecone', async (importOriginal) => {
  const original = await importOriginal<typeof import('@pinecone-database/pinecone')>()
  const { mockObject } = await import('vitest/mocker')

  const mocked = mockObject(
    { type: 'automock', spyOn: vi.spyOn },
    original
  )

  return mocked
})
```

✅ Damit umgehst du den rekursiven Import, weil `importOriginal` speziell von Vitest bereitgestellt wird, um **das Originalmodul** zu importieren – **nicht das bereits gemockte.**

---

## 🧪 Wann kannst du `importMock()` **sicher** verwenden?

Nur **außerhalb von `vi.mock()`**, z. B. in deinem Testcode selbst:

```ts
import { importMock } from 'vitest/mocker'

test('something', async () => {
  const pinecone = await importMock('@pinecone-database/pinecone')
  // Test mit gemocktem Modul
})
```

Oder innerhalb eines **anderen Moduls**, das nicht am selben Mock beteiligt ist.

---

## ✅ Empfehlung

Wenn du innerhalb von `vi.mock()` bist:

* ❌ **Kein `importMock()` verwenden**
* ✅ Nutze `importOriginal` + `mockObject`

Wenn du das Modul **extern mocken** willst (z. B. für Helper oder generische Testmocks), dann:

* ✅ `importMock()` verwenden – aber **nie** aus dem gleichen Modulkontext.


</details>




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
