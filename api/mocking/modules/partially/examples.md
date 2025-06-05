# Rule: Effective Mocking of Modules/NPM-Packages in Vitest

## Overview

This documentation outlines the preferred and complementary approaches to mocking external modules/NPM packages in Vitest tests.

## ❗ Critical Instructions

* **PREFERRED APPROACH (Modular):** Use a **modular mock structure** with a dedicated mock factory class inside `__mocks__/`, to centrally manage and import mock logic in test files. This avoids hoisting issues, prevents unbound method errors, improves reusability, and increases maintainability.
* **SECONDARY APPROACH (Full with `mockObject`):** If the modular approach isn’t feasible, use `vi.mock` in combination with `await importOriginal()` and `mockObject` from `vitest/mocker` to create a **complete, automatically generated mock object** of the module. This is type-safe and avoids having to manually replicate all exports.
* **COMPLEMENTARY APPROACH (Partial):** If only specific functions of a module need to be overridden and a full mock isn't practical, use `vi.mock` with a factory function that calls `await vi.importActual()` to load the original module and selectively override functions with `vi.fn()`. This approach should only be used when the preferred methods are not applicable.
* **AVOID** using `importMock()` inside a `vi.mock()` factory, as it can cause infinite recursion.
* Ensure that mocked functions simulate the expected behavior for your tests.
* Use `vi.mocked` for typed mock references when working with partial mocks.

## ✅ Examples

### ⭐ Preferred Approach: Modular Mock Structure

This approach uses a separate mock factory file to encapsulate and reuse mock logic.

#### 1. Mock Factory File (`__mocks__/axios.ts`)

```typescript
/* eslint-disable func-names */
/* eslint-disable @typescript-eslint/naming-convention */
/* eslint-disable @typescript-eslint/prefer-readonly-parameter-types */

// ==== DEPENDENCIES ====
import type { AxiosRequestConfig, HttpStatusCode } from 'axios'
import { vi } from 'vitest'

const actualAxios = await vi.importActual<typeof import('axios')>('axios')

// Helper function to create a mock response
const createMockResponse = <T>(data: T, config?: Readonly<AxiosRequestConfig>) => ({
    data,
    status: 200,
    statusText: 'OK',
    headers: {},
    config: config ?? {},
    request: {}
})

// Mock the default axios instance
const mockedAxios = {
    ...actualAxios.default,
    get: vi.fn(async function get(url: string, config?: AxiosRequestConfig) {
        if (url.includes('/api/partner/privyou-tool/version')) {
            return createMockResponse({ success: true, version: 31 }, config)
        }

        if (url.includes('/api/partner/privyou-tool/config')) {
            return createMockResponse({ success: true, pvsConfig: { updatedAt: new Date().toISOString() } }, config)
        }

        if (url.includes('/api/partner/practice/customers')) {
            return createMockResponse({ success: true, data: 'mockEncryptedData' }, config)
        }

        const urlWithBaseUrl = `${this.defaults.baseURL ?? ''}${url}`
        return actualAxios.default.get(urlWithBaseUrl, config)
    }),

    post: vi.fn(async function post(url: string, data?: unknown, config?: AxiosRequestConfig) {
        if (url.includes('/api/partner/practice/finding')) {
            return createMockResponse({ success: true, finding: 'mockFinding' }, config)
        }

        if (url.includes('/api/login-privyou')) {
            return createMockResponse({
                success: true,
                user: { name: 'testuser', groups: ['testgroup'] },
                token: 'testtoken'
            }, config)
        }

        const urlWithBaseUrl = `${this.defaults.baseURL ?? ''}${url}`
        return actualAxios.default.post(urlWithBaseUrl, data, config)
    })

    // put: vi.fn(async(url: string, data?: unknown, config?: AxiosRequestConfig) => {
    //     return Promise.resolve(createMockResponse({}, config))
    // }),

    // patch: vi.fn(async(url: string, data?: unknown, config?: AxiosRequestConfig) => {
    //     return Promise.resolve(createMockResponse({}, config))
    // }),

    // delete: vi.fn(async(url: string, config?: AxiosRequestConfig) => {
    //     return Promise.resolve(createMockResponse({}, config))
    // }),

    // head: vi.fn(async(url: string, config?: AxiosRequestConfig) => {
    //     return Promise.resolve(createMockResponse({}, config))
    // }),

    // options: vi.fn(async(url: string, config?: AxiosRequestConfig) => {
    //     return Promise.resolve(createMockResponse({}, config))
    // }),

    // // Generic request method
    // request: vi.fn(async(config: AxiosRequestConfig) => {
    //     return Promise.resolve(createMockResponse({}, config))
    // }),

    // // Convenience methods for creating new instances
    // create: vi.fn((config?: AxiosRequestConfig) => ({
    //     ...mockedAxios,
    //     defaults: { ...actualAxios.default.defaults, ...config }
    // })),

    // // Interceptors (simplified)
    // interceptors: {
    //     request: {
    //         use: vi.fn(() => 0),
    //         eject: vi.fn(),
    //         clear: vi.fn(),
    //         forEach: vi.fn()
    //     },
    //     response: {
    //         use: vi.fn(() => 0),
    //         eject: vi.fn(),
    //         clear: vi.fn(),
    //         forEach: vi.fn()
    //     }
    // },

    // // Default configuration
    // defaults: {
    //     ...actualAxios.default.defaults,
    //     timeout: 0,
    //     withCredentials: false,
    //     responseType: 'json' as const,
    //     maxContentLength: -1,
    //     maxBodyLength: -1,
    //     maxRedirects: 21
    // },

    // // Utility methods
    // getUri: vi.fn((config?: AxiosRequestConfig): string => {
    //     return config?.url ?? ''
    // }),

    // // Static methods from axios module
    // isCancel: actualAxios.isCancel,
    // isAxiosError: actualAxios.isAxiosError,
    // toFormData: actualAxios.toFormData,
    // formToJSON: actualAxios.formToJSON,
    // all: vi.fn(async <T>(values: (T | Promise<T>)[]): Promise<T[]> => {
    //     return Promise.all(values)
    // }),
    // spread: actualAxios.spread,
    // AxiosError: actualAxios.AxiosError,
    // AxiosHeaders: actualAxios.AxiosHeaders,
    // HttpStatusCode: actualAxios.HttpStatusCode
}

export default mockedAxios
```

#### 2. Simplified Test Setup (`test/unit/test-setup.ts`)

```typescript
/**
 * 📌 test/unit/test-setup.ts
 *
 * Setup file for unit tests.
 * This file is loaded via the setupFiles configuration in vitest.unit.config.ts.
 *
 * IMPORTANT: vi.* functions can be used here because setupFiles
 * run in the test suite context.
 */

import { vi } from 'vitest'

function setupUnitTestEnvironment(): void {
    console.info('🧪 Initializing unit test environment...')
    
    vi.stubGlobal('UNIT_TEST_MODE', true)

    // Mock Axios for HTTP requests (__mocks__/axios.ts)
    vi.mock('axios')

    console.info('✅ Unit test environment successfully initialized')
}

// Auto-execute on import
setupUnitTestEnvironment()
```

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
