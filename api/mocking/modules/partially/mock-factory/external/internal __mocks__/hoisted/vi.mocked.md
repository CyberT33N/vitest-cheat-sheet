# Hoisted
- Das nachfolgende Beispiel mockt **Axios** in der **Test-Setup-Datei**, welche mit **Vitest** vor den Tests geladen wird. Das Problem ist, dass es Komplikationen geben kann, wie z.B. wenn **FS** benutzt wird und innerhalb vom Code es an irgendeiner Stelle referenziert wird, weil durch den **Hoisted-Mechanismus** das Modul zu diesem Zeitpunkt schon überschrieben wird und das Modul dann nicht mehr verfügbar ist. Das bedeutet, diese Technik **MUSS** mit Bedacht benutzt werden, da zu diesem Zeitpunkt das Modul komplett überschrieben wird.


`__mocks__/axios.ts`

```typescript
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

// Mock the default axios instance
const mockedAxios = {
    ...actualAxios.default,
    get: vi.fn(async function get(
        this: typeof actualAxios.default, url: string, config?: AxiosRequestConfig
    ) {
        // if (url.includes('/api/partner/privyou-tool/version')) {
        //     return createMockResponse({
        //         success: true,
        //         version: 31
        //     }, config)
        // }

        // if (url.includes('/api/partner/privyou-tool/config')) {
        //     return createMockResponse({
        //         success: true,
        //         pvsConfig: {
        //             updatedAt: new Date().toISOString()
        //         }
        //     }, config)
        // }

        // if (url.includes('/api/partner/practice/customers')) {
        //     return createMockResponse({
        //         success: true,
        //         data: 'mockEncryptedData'
        //     }, config)
        // }

        // const urlWithBaseUrl = `${this.defaults.baseURL ?? ''}${url}`
        // return Promise.resolve(actualAxios.default.get(urlWithBaseUrl, config))

        return createMockResponse({
            success: true,
            data: 'mockEncryptedData'
        }, config)
    }),

    post: vi.fn(async function post(
        this: typeof actualAxios.default, url: string, data?: unknown, config?: AxiosRequestConfig
    ) {
        if (url.includes('/api/partner/practice/finding')) {
            return Promise.resolve(createMockResponse({
                success: true,
                finding: 'mockFinding'
            }, config))
        }

        if (url.includes('/api/partner/practice/customer')) {
            return Promise.resolve(createMockResponse({
                success: true,
                finding: 'mockFinding'
            }, config))
        }
    
        if (url.includes('/api/login-privyou')) {
            return Promise.resolve(createMockResponse({
                success: true,
                user: { name: 'testuser', groups: ['testgroup'] },
                token: 'testtoken'
            }, config))
        }
    
        const urlWithBaseUrl = `${this.defaults.baseURL ?? ''}${url}`
        return Promise.resolve(actualAxios.default.post(urlWithBaseUrl, data, config))
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


<br><br>

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








<br><br>

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

    // Mock Axios for HTTP requests (__mocks__/axios.ts)
    vi.mock('axios')

    console.info('✅ Unit test environment successfully initialized')
}

// Auto-execute on import
setupUnitTestEnvironment()
```
- Wenn du **NICHT WILLST**, dass vor jedem einzelnen **Test** durch die **Test-Setup-Datei** gemockt wird, **KANNST** du es natürlich auch nur spezifisch in der **Testdatei** aufrufen. `vi.mock('axios')`




<br><br>


test/utils/vitest/mocking.ts
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

<br><br>


`test.ts`
```typescript
import axios, { type AxiosResponse } from 'axios'
import { describe, it, expect, vi, afterEach, MockInstance } from 'vitest'
import { userService } from '../../src/userService.ts'
import { createMockedModule } from '@test/utils/vitest/mocking.ts'

describe('UserService', () => {
    // ✅ ENTERPRISE PATTERN: Deep mocking für vollständige Mock-Kontrolle  
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
    // 🚀 ENTERPRISE PATTERN: Typsichere Mock-Überschreibung ohne ANY-Casting
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










