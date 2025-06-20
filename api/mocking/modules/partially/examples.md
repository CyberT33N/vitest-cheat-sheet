









































### Secondary Approach: Full Automatic Mocking with `mockObject`

This is the **fallback** if the modular approach isn’t viable. It’s type-safe and avoids manually mocking each export, but it doesn’t encapsulate the mock logic as cleanly.

```typescript
// test/my-service.test.ts
import { describe, it, expect, vi, beforeEach, type MockedObject } from 'vitest';
import { mockObject } from 'vitest/mocker';
import { PineconeService } from './pinecone-service';
import env from '../../src/env';

type PineconeModule = typeof import('@pinecone-database/pinecone');
type MockedPineconeModule = MockedObject<PineconeModule>;

const mockFactory = vi.hoisted(() => {
  let mockedModule: MockedPineconeModule;

  const createAndStoreMockedModule = async (): Promise<MockedPineconeModule> => {
    const original = await vi.importActual<PineconeModule>('@pinecone-database/pinecone');
    const module = mockObject(
      {
        type: 'automock',
        spyOn: vi.spyOn,
        globalConstructors: { Object, Function, RegExp, Array, Map }
      },
      original
    ) as MockedPineconeModule;

    mockedModule = module;
    return module;
  };

  return {
    getMockedPineconeModule: (): MockedPineconeModule => mockedModule,
    createAndStoreMockedModule
  };
});

vi.mock('@pinecone-database/pinecone', async () => {
  return mockFactory.createAndStoreMockedModule();
});

const createStandardPineconeService = (): PineconeService => {
  return new PineconeService({
    apiKey: env.PINECONE_API_KEY,
    namespace: env.PINECONE_RULES_NAMESPACE
  });
};

describe('PineconeService', () => {
  let service: PineconeService;
  let mockedPinecone: MockedPineconeModule;

  beforeEach(() => {
    mockedPinecone = mockFactory.getMockedPineconeModule();
    service = createStandardPineconeService();
  });

  it('✅ should initialize with correct API key and namespace', () => {
    expect(mockedPinecone.Pinecone).toHaveBeenCalledWith({ apiKey: env.PINECONE_API_KEY });
    expect(Reflect.get(service, '_namespace')).toBe(env.PINECONE_RULES_NAMESPACE);
  });
});
```
































### Complementary Approach: Partial Mocking (Only if Necessary)

Use when only specific functions of a module need to be mocked and a full mock is overkill or impractical.

#### Example: `@google/genai`

```typescript
import * as googleGenAi from '@google/genai';
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { GoogleEmbeddingService } from './google-embedding-service';
import env from '../../src/env';

const mockProvider = vi.hoisted(() => {
    const mockEmbedContent = vi.fn();
    const mockGetGenerativeModel = vi.fn(() => ({}));

    const mockGoogleGenAI = vi.fn(() => ({
        getGenerativeModel: mockGetGenerativeModel,
        models: {
            embedContent: mockEmbedContent
        }
    }));

    return {
        mockEmbedContent,
        mockGetGenerativeModel,
        mockGoogleGenAI
    };
});

vi.mock('@google/genai', async () => {
    const actualPackage = await vi.importActual<typeof googleGenAi>('@google/genai');
    const { mockGoogleGenAI } = mockProvider;
    return {
        ...actualPackage,
        GoogleGenAI: mockGoogleGenAI
    };
});

describe('GoogleEmbeddingService()', () => {
    let service: GoogleEmbeddingService;

    beforeEach(() => {
        vi.stubEnv('GEMINI_API_KEY', env.GEMINI_API_KEY);
        service = new GoogleEmbeddingService();
    });

    describe('Constructor', () => {
        it('should initialize successfully with default values', () => {
            expect(service).toBeInstanceOf(GoogleEmbeddingService);
            expect(Reflect.get(service, '_apiKey')).toBe(env.GEMINI_API_KEY);
            expect(mockProvider.mockGoogleGenAI).toHaveBeenCalledWith(expect.objectContaining({
                apiKey: env.GEMINI_API_KEY
            }));
        });
    });
});
```

#### Example: Direct Mocking (`fs/promises`)

```typescript
import * as fsPromises from 'fs/promises';
import { vi, describe, it, expect } from 'vitest';

vi.mock('fs/promises', () => ({
  mkdir: vi.fn(),
  writeFile: vi.fn(),
  readFile: vi.fn(),
  readdir: vi.fn(),
}));

const mockedFsPromises = vi.mocked(fsPromises, true);

describe('Filesystem Operations', () => {
  it('should write a file', async () => {
    const filePath = 'test/output.txt';
    const fileContent = 'Hello world!';
    mockedFsPromises.writeFile.mockResolvedValue(undefined);
    await fsPromises.writeFile(filePath, fileContent);
    expect(mockedFsPromises.writeFile).toHaveBeenCalledWith(filePath, fileContent);
  });
});
```

## 🤔 Rationale

Mocking modules is essential for isolated unit tests.

* The **modular mock structure** is the **preferred approach**, as it centralizes mock logic, keeps test files clean, maximizes reusability, and avoids common issues like unbound method errors and hoisting problems.
* The **SECONDARY APPROACH (Full with `mockObject`)** is a strong alternative if modular mocking is impractical. It covers the entire module API and ensures type safety.
* The **COMPLEMENTARY APPROACH (Partial)** is useful when only a few specific functions of a large module need mocking, but requires more manual effort and is less type-safe.
* Avoiding `importMock()` inside `vi.mock()` prevents recursion issues and ensures mocks initialize correctly.

## 🧠 Recap

| Action                                | Context             | Status                                |
| ------------------------------------- | ------------------- | ------------------------------------- |
| ❌ `importMock()`                      | Inside `vi.mock()`  | ❌ Never (Infinite Recursion)          |
| ✅ Use modular mock factory wrapper    | Anywhere            | ⭐ **Preferred for complex scenarios** |
| ✅ `importOriginal()` + `mockObject()` | Inside `vi.mock()`  | ✅ Secondary for full mocking          |
| ✅ `importActual()` + manual mocks     | Inside `vi.mock()`  | ✅ Complementary for partial mocking   |
| ✅ `importMock()`                      | Outside `vi.mock()` | ✅ Safe (for already mocked modules)   |

Mock smart. Mock safe.
