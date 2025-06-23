# Enterprise Mock Pattern - Universelle Mock-Architektur

Das **Enterprise Mock Pattern** ist der **empfohlene Standard** für alle Mock-Dateien in Vitest-Projekten. Dieses Pattern bietet universelle Kompatibilität für verschiedene Import-Stile und vermeidet die Probleme des hoisted-Patterns.

## Warum Enterprise Pattern?

- ✅ **Universelle Kompatibilität:** Funktioniert für beide Import-Stile (`default` und `named exports`)
- ✅ **Keine Hoisting-Probleme:** Vermeidet Race Conditions und Module-Überschreibungen
- ✅ **Einheitliche Architektur:** Ein Pattern für alle Mock-Dateien
- ✅ **Bessere Typsicherheit:** Vollständige TypeScript-Unterstützung
- ✅ **Einfache Maintenance:** Klare, vorhersagbare Struktur

## Pattern-Struktur

```typescript
// 1. Named exports für alle Mock-Funktionen
export const functionName = vi.fn(/* implementation */)

// 2. Default export mit allen named exports für Kompatibilität
const mockFactory = {
    ...actualModule,
    functionName,
    // ... alle anderen Funktionen
}

export default mockFactory
```

## Beispiel: Axios Mock

`__mocks__/axios.ts`

```typescript
/* eslint-disable @typescript-eslint/naming-convention */
/* eslint-disable func-names */
/* eslint-disable @typescript-eslint/prefer-readonly-parameter-types */

// ==== DEPENDENCIES ====
import type { AxiosRequestConfig, HttpStatusCode } from 'axios'
import { vi } from 'vitest'

const actualAxios = await vi.importActual<typeof import('axios')>('axios')

// Helper function to create a mock response
const createMockResponse = <T>(data: T, config?: Readonly<AxiosRequestConfig>): {
    data: T;
    status: HttpStatusCode;
    statusText: string;
    headers: Record<string, unknown>;
    config: AxiosRequestConfig;
    request: Record<string, unknown>;
} => ({
        data,
        status: 200,
        statusText: 'OK',
        headers: {},
        config: config ?? {},
        request: {}
    })

// ✅ ENTERPRISE PATTERN: Named exports für alle Mock-Funktionen
export const get = vi.fn(async function get(
    url: string, config?: AxiosRequestConfig
) {
    console.info('axios.get called:', url)
    return Promise.resolve(createMockResponse({
        success: true,
        data: 'mockEncryptedData'
    }, config))
})

export const post = vi.fn(async function post(
    url: string, data?: unknown, config?: AxiosRequestConfig
) {
    console.info('axios.post called:', url)
    
    if (url.includes('/api/partner/practice/finding')) {
        return Promise.resolve(createMockResponse({
            success: true,
            finding: 'mockFinding'
        }, config))
    }

    if (url.includes('/api/partner/practice/customer')) {
        return Promise.resolve(createMockResponse({
            success: true,
            customer: 'mockCustomer'
        }, config))
    }

    if (url.includes('/api/login-privyou')) {
        return Promise.resolve(createMockResponse({
            success: true,
            user: { name: 'testuser', groups: ['testgroup'] },
            token: 'testtoken'
        }, config))
    }

    return Promise.resolve(createMockResponse({
        success: true,
        data: 'mockPostData'
    }, config))
})

export const put = vi.fn(async function put(
    url: string, data?: unknown, config?: AxiosRequestConfig
) {
    console.info('axios.put called:', url)
    return Promise.resolve(createMockResponse({
        success: true,
        updated: true
    }, config))
})

export const patch = vi.fn(async function patch(
    url: string, data?: unknown, config?: AxiosRequestConfig
) {
    console.info('axios.patch called:', url)
    return Promise.resolve(createMockResponse({
        success: true,
        patched: true
    }, config))
})

export const deleteMethod = vi.fn(async function deleteMethod(
    url: string, config?: AxiosRequestConfig
) {
    console.info('axios.delete called:', url)
    return Promise.resolve(createMockResponse({
        success: true,
        deleted: true
    }, config))
})

export const head = vi.fn(async function head(
    url: string, config?: AxiosRequestConfig
) {
    console.info('axios.head called:', url)
    return Promise.resolve(createMockResponse({}, config))
})

export const options = vi.fn(async function options(
    url: string, config?: AxiosRequestConfig
) {
    console.info('axios.options called:', url)
    return Promise.resolve(createMockResponse({}, config))
})

export const request = vi.fn(async function request(config: AxiosRequestConfig) {
    console.info('axios.request called:', config.url)
    return Promise.resolve(createMockResponse({
        success: true,
        data: 'mockRequestData'
    }, config))
})

export const create = vi.fn(function create(config?: AxiosRequestConfig) {
    console.info('axios.create called with config:', config)
    return {
        ...mockedAxios,
        defaults: { ...actualAxios.default.defaults, ...config }
    }
})

// ✅ ENTERPRISE PATTERN: Default export mit allen Named exports für Kompatibilität
const mockedAxios = {
    ...actualAxios.default,
    
    // Include all named exports in the default export for compatibility
    get,
    post,
    put,
    patch,
    delete: deleteMethod, // 'delete' ist ein reserved keyword, daher alias
    head,
    options,
    request,
    create,

    // Interceptors (simplified)
    interceptors: {
        request: {
            use: vi.fn(() => 0),
            eject: vi.fn(),
            clear: vi.fn(),
            forEach: vi.fn()
        },
        response: {
            use: vi.fn(() => 0),
            eject: vi.fn(),
            clear: vi.fn(),
            forEach: vi.fn()
        }
    },

    // Default configuration
    defaults: {
        ...actualAxios.default.defaults,
        timeout: 0,
        withCredentials: false,
        responseType: 'json' as const,
        maxContentLength: -1,
        maxBodyLength: -1,
        maxRedirects: 21
    },

    // Utility methods
    getUri: vi.fn((config?: AxiosRequestConfig): string => {
        return config?.url ?? ''
    }),

    // Static methods from axios module
    isCancel: actualAxios.isCancel,
    isAxiosError: actualAxios.isAxiosError,
    toFormData: actualAxios.toFormData,
    formToJSON: actualAxios.formToJSON,
    all: vi.fn(async <T>(values: (T | Promise<T>)[]): Promise<T[]> => {
        return Promise.all(values)
    }),
    spread: actualAxios.spread,
    AxiosError: actualAxios.AxiosError,
    AxiosHeaders: actualAxios.AxiosHeaders,
    HttpStatusCode: actualAxios.HttpStatusCode
}

export default mockedAxios
```

## Service Beispiel

`userService.ts`

```typescript
import axios from 'axios'

export class UserService {
  async getUser(id) {
    const response = await axios.get(`https://api.example.com/users/${id}`)
    return response.data
  }

  async createUser(userData) {
    const response = await axios.post('https://api.example.com/users', userData)
    return response.data
  }

  async deleteUser(id) {
    await axios.delete(`https://api.example.com/users/${id}`)
    return { success: true }
  }
}

export const userService = new UserService()
```

## Test Setup (vereinfacht)

`test/unit/test-setup.ts`

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

    vi.mock('axios')
    vi.mock('windows-drive')

    console.info('✅ Unit test environment successfully initialized')
}

// Auto-execute on import
setupUnitTestEnvironment()
```
- Wenn du **NICHT WILLST**, dass vor jedem einzelnen **Test** durch die **Test-Setup-Datei** gemockt wird, **KANNST** du es natürlich auch nur spezifisch in der **Testdatei** aufrufen. `vi.mock('axios')`



## Vitest Helper (unverändert)

`test/utils/vitest/ts-helper.ts`

```typescript
import { type MockInstance } from 'vitest'

// 🚀 ULTIMATIVE LÖSUNG: DeepMocked Type mit direkter MockInstance Integration
// MockInstance hat mockReturnValue, mockImplementation, mockRestore, etc.
export type DeepMocked<T> = {
  readonly [K in keyof T]: T[K] extends (...args: infer A) => infer R
      ? MockInstance<(...args: A) => R> & ((...args: A) => R)
      : T[K] extends object
      ? DeepMocked<T[K]>
      : T[K]
}

// 🎯 HELPER FUNCTION: Semantisch klares Single-Cast ohne doppelte Assertion
export const createMockedModule = <T>(mockedModule: T): DeepMocked<T> => 
  mockedModule as DeepMocked<T>
```

## Test Implementation

`test.ts`

```typescript
import axios, { type AxiosResponse } from 'axios'
import { describe, it, expect, vi, afterEach } from 'vitest'
import { userService } from '../../src/userService.ts'
import { createMockedModule, type DeepMocked } from '@test/utils/vitest/ts-helper.ts'

describe('UserService', () => {
    // ✅ Deep mocking für vollständige Mock-Kontrolle  
    const viMockedAxios = vi.mocked(axios, { deep: true })
    const mockedAxios = createMockedModule(viMockedAxios)

    afterEach(() => {
        // Mock ist bereits korrekt typisiert - kein erneutes Assignment nötig
        vi.clearAllMocks()
    })

    it('sollte einen Benutzer abrufen (Auto-Mock)', async() => {
        const user = await userService.getUser(1)
    
        // ✅ Das funktioniert - wir bekommen die gemockten Daten
        expect(user).toEqual({
            success: true,
            data: 'mockEncryptedData'
        })

        // ✅ KEIN .default nötig - axios ist direkt die AxiosStatic-Instanz
        expect(mockedAxios.get).toHaveBeenCalledWith('https://api.example.com/users/1')
        expect(mockedAxios.get).toHaveBeenCalledTimes(1)
    })

    it('sollte einen Benutzer abrufen (CUSTOM INLINE MOCK)', async() => {
        const customResponse: AxiosResponse = {
            data: {
                success: true,
                data: 'CUSTOM_INLINE_MOCK_DATA',
                customField: 'NUR_FÜR_DIESEN_TEST'
            },
            status: 200,
            statusText: 'OK',
            headers: {},
            config: {
                headers: {} as any // Minimal config für Mock-Zwecke
            },
            request: {}
        }
    
        // ✅ Deep Mocking ermöglicht typsichere Mock-Methoden
        mockedAxios.get.mockResolvedValueOnce(customResponse)

        const user = await userService.getUser(1)
    
        // ✅ Jetzt bekommen wir die CUSTOM-DATEN statt der hardcodierten Mock-Factory-Daten
        expect(user).toEqual({
            success: true,
            data: 'CUSTOM_INLINE_MOCK_DATA',
            customField: 'NUR_FÜR_DIESEN_TEST'
        })

        expect(mockedAxios.get).toHaveBeenCalledWith('https://api.example.com/users/1')
        expect(mockedAxios.get).toHaveBeenCalledTimes(1)
    })
})
```

## Vorteile des Enterprise Patterns

1. **Universelle Kompatibilität:** Sowohl `import axios from 'axios'` als auch `import { get } from 'axios'` funktionieren
2. **Hoisting-sicher:** Keine Race Conditions oder Module-Überschreibungsprobleme
3. **Einheitliche Architektur:** Ein Pattern für alle Mock-Szenarien
4. **Bessere Developer Experience:** Vollständige IntelliSense und Typsicherheit
5. **Einfache Maintenance:** Klare Struktur und vorhersagbares Verhalten