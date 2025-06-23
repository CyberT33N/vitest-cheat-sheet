# Runtime

## Example

- Dieses **Beispiel** ist das **Äquivalent** zum **Hoisted-Beispiel**. Hier laden wir in der **Runtime** erst den **Mog**. Die **Begründung** ist, dass gegebenenfalls in einem **Test-Setup** oder an anderen Stellen schon **Module** aufgerufen werden, welche wir **IMPLEMENTIEREN** zu **mocken**. Um dementsprechend **Konflikte** zu vermeiden, starten wir den **Mock** erst innerhalb der **Tests**.

- Damit folgender Ansatz funktioniert, **MÜSSEN** wir `doMock` verwenden und es in `BeforeAll` initialisieren, also zur **RUNTIME**. Das **PROBLEM** ist, dass der jeweilige **SERVICE**, den wir testen, also die **KLASSE** oder die **FUNKTION**, nicht statisch oben als **TOP-LEVEL-IMPORT** benutzt werden kann. Wir **MÜSSEN** zuerst `doMock` verwenden und **MÜSSEN** danach sicherstellen, dass wir das **MODUL** importieren. Es **DARF NICHT** sein, dass ein **MODUL** als **TOP-LEVEL-IMPORT** importiert wird, nur um z.B. die **TYPES** usw. zu beziehen, weil es dann nicht mehr funktioniert. Es **MUSS** sichergestellt werden, dass der allererste **IMPORT** von dem **SERVICE**, den wir später testen wollen, mit dem **MOCK** nach `doMock` passiert.



`__mocks__/axios.ts`

```typescript
/* eslint-disable func-names */
/* eslint-disable @typescript-eslint/prefer-readonly-parameter-types */
/*
 ▄▄▄·▄▄▄  ▪   ▌ ▐· ▄▄▄· ·▄▄▄▄  ▄▄▄ . ▐ ▄ ▄▄▄▄▄
▐█ ▄█▀▄ █·██ ▪█·█▌▐█ ▀█ ██▪ ██ ▀▄.▀·•█▌▐█•██  
 ██▀·▐▀▀▄ ▐█·▐█▐█•▄█▀▀█ ▐█· ▐█▌▐▀▀▪▄▐█▐▐▌ ▐█.▪
▐█▪·•▐█•█▌▐█▌ ███ ▐█ ▪▐▌██. ██ ▐█▄▄▌██▐█▌ ▐█▌·
.▀   .▀  ▀▀▀▀. ▀   ▀  ▀ ▀▀▀▀▀•  ▀▀▀ ▀▀ █▪ ▀▀▀ 
 * © privadent GmbH. All rights reserved.  
 * Unauthorized copying, distribution, or modification of this software is strictly prohibited.  
*/

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


`test.ts`
```typescript
import axios from 'axios'
import { describe, it, expect, vi, afterEach, beforeAll, type MockInstance } from 'vitest'
// ✅ FAZIT: Statischer Import funktioniert NICHT mit vi.resetModules() + vi.doMock()

// 🚀 ULTIMATIVE LÖSUNG: DeepMocked Type mit direkter MockInstance Integration
// MockInstance hat mockReturnValue, mockImplementation, mockRestore, etc.
type DeepMocked<T> = {
    readonly [K in keyof T]: T[K] extends (...args: infer A) => infer R
        ? MockInstance<(...args: A) => R> & ((...args: A) => R)
        : T[K] extends object
        ? DeepMocked<T[K]>
        : T[K]
}

// 🎯 HELPER FUNCTION: Semantisch klares Single-Cast ohne doppelte Assertion
const createMockedModule = <T>(mockedModule: T): DeepMocked<T> => 
    mockedModule as DeepMocked<T>

describe('UserService', () => {
    // ✅ PERFEKTE LÖSUNG: Rekursiver Utility Type ersetzt ALLE Methoden
    // Funktioniert mit JEDEM Package - axios, fs, path, etc.
    let mockedAxios: DeepMocked<typeof axios>
    let UserService: typeof import('@/main/services/dampsoft/DampsoftService.ts')
    let userService: InstanceType<typeof UserService.userService>

    beforeAll(async() => {
        // ✅ ZURÜCK ZUR FUNKTIONIERENDEN LÖSUNG: vi.doMock() + dynamische Imports
        vi.doMock('axios')
        
        // Nach doMock müssen wir das Modul dynamisch importieren
        const axiosModule = await import('axios')
        const viMockedAxios = vi.mocked(axiosModule.default, { deep: true })
        mockedAxios = createMockedModule(viMockedAxios)
        
        // userService NACH axios-Mock importieren
        const userServiceModule = await import('@/userService.ts')
        userService = userServiceModule.userService
    })

    afterEach(() => {
        vi.clearAllMocks()
    })

    it('sollte einen Benutzer abrufen (Auto-Mock)', async() => {
        mockedAxios.get.mockReturnValue(Promise.resolve({
            data: {
                success: true,
                data: 'mockEncryptedData'
            }
        }))

        // ✅ FUNKTIONIERT: userService verwendet jetzt gemocktes axios
        const user = await userService.getUser(1)
    
        // ✅ Das funktioniert - wir bekommen die gemockten Daten
        expect(user).toEqual({
            success: true,
            data: 'mockEncryptedData'
        })

        // ✅ FIX: Nach vi.mock() sind alle Methoden automatisch Mock-Funktionen
        // Kein unbound-method Problem, da Vitest die Bindung übernimmt
        expect(mockedAxios.get).toHaveBeenCalledWith('https://api.example.com/users/1')
        expect(mockedAxios.get).toHaveBeenCalledTimes(1)
    })
})

```
- Es ist **SEHR WICHTIG** zu beachten, dass bei dieser Variante **KEIN** statischer Import oben vorhanden sein **DARF**. Sobald wir anfangen, den **Service** zu importieren, den wir testen **WOLLEN**, in dem Fall den **User Service**, würden wir ein **PROBLEM** kriegen und der **Mod** würde nicht funktionieren.






