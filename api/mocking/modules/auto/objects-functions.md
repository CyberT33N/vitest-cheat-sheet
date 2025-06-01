# Auto Mocking


## Objects/Functions

<details><summary>Click to expand..</summary>

### ❌ Problem: `importMock()` Inside `vi.mock()` = 💥 Infinite Recursion

Calling `importMock()` **inside** a `vi.mock()` block is a trap:
It tries to load the very module you're currently mocking → triggers `vi.mock()` again → **infinite loop** → boom.

---

### ✅ Solution #1: Use `importOriginal()`

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

### ✅ Solution #2: Mock Factory Pattern - `importActual()`

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


