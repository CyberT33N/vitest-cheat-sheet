# 🏗️ Enterprise Test Context Architecture - AI Agent Reference Guide

## 📋 Architektur-Übersicht

Diese Regel definiert eine **zentrale, typisierte Test-Context-Management-Architektur** für Enterprise-TypeScript-Projekte. Die Architektur eliminiert Boilerplate-Code und ermöglicht saubere Trennung von Test-Modulen.

### Komponenten
1. **TestContextManager** - Zentrale Singleton-Registry für Test-Kontexte
2. **Parent Test File** - Haupttest-Datei mit Kontext-Setup und Mock-Factories
3. **Sub Test Files** - Modulare Test-Dateien die den Kontext konsumieren

---

## 🎯 1. TestContextManager (Central Utility)

**Pfad:** `test/utils/TestContextManager.ts`

```typescript
/**
 * 🏗️ **Central Enterprise Test Context Manager**
 * 
 * A generic, centralized test context management system that can be used
 * across all test files in the project. Uses a registry pattern with unique
 * context keys to manage multiple test contexts simultaneously.
 * 
 * **Enterprise Benefits:**
 * ✅ Single source of truth for test context management
 * ✅ Type-safe generics for different test context interfaces
 * ✅ Scalable across all service tests
 * ✅ Clean separation of concerns
 * ✅ Easy to maintain and extend
 * ✅ Supports parallel test execution
 * 
 * **Usage Example:**
 * ```typescript
 * // In main test file:
 * const contextKey = 'DampsoftPatientService'
 * const manager = TestContextManager.getInstance<ITestContext>()
 * manager.setContext(contextKey, context)
 * 
 * // In subtest files:
 * const context = useTestContext<ITestContext>('DampsoftPatientService')
 * ```
 */
export class TestContextManager {
    private static _instance: TestContextManager | undefined
    private readonly _contexts = new Map<string, unknown>()

    private constructor() {
        // Private constructor for singleton pattern
    }

    /**
     * Get the singleton instance of TestContextManager
     */
    public static getInstance(): TestContextManager {
        TestContextManager._instance ??= new TestContextManager()
        return TestContextManager._instance
    }

    /**
     * Set context for a specific test service
     * @param contextKey - Unique identifier for the test service (e.g., 'DampsoftPatientService')
     * @param context - The test context object
     */
    // eslint-disable-next-line @typescript-eslint/no-unnecessary-type-parameters
    public setContext<TContext>(contextKey: string, context: TContext): void {
        this._contexts.set(contextKey, context)
    }

    /**
     * Get context for a specific test service
     * @param contextKey - Unique identifier for the test service
     * @throws Error if context not found
     */
    // eslint-disable-next-line @typescript-eslint/no-unnecessary-type-parameters
    public getContext<TContext>(contextKey: string): TContext {
        const context = this._contexts.get(contextKey) as TContext
        
        if (context === undefined) {
            throw new Error(
                `❌ TestContext '${contextKey}' not initialized! ` +
                `Make sure the main test file calls setContext('${contextKey}', context) in beforeEach.` +
                `\n\nAvailable contexts: [${Array.from(this._contexts.keys()).join(', ')}]`
            )
        }
        
        return context
    }

    /**
     * Clear context for a specific test service
     * @param contextKey - Unique identifier for the test service
     */
    public clearContext(contextKey: string): void {
        this._contexts.delete(contextKey)
    }

    /**
     * Clear all contexts (useful for global test cleanup)
     */
    public clearAllContexts(): void {
        this._contexts.clear()
    }

    /**
     * Check if context exists for a specific test service
     * @param contextKey - Unique identifier for the test service
     */
    public hasContext(contextKey: string): boolean {
        return this._contexts.has(contextKey)
    }

    /**
     * Get all registered context keys
     */
    public getContextKeys(): string[] {
        return Array.from(this._contexts.keys())
    }

    /**
     * Get the number of registered contexts
     */
    public getContextCount(): number {
        return this._contexts.size
    }
}

/**
 * 🎯 **Convenience Hook for Subtest Files**
 * 
 * Provides easy access to test context with proper error handling
 * and type safety. This is the main API that subtest files should use.
 * 
 * @param contextKey - Unique identifier for the test service
 * @returns The typed test context
 * 
 * @example
 * ```typescript
 * // In DampsoftPatientService subtest files:
 * const context = useTestContext<ITestContext>('DampsoftPatientService')
 * 
 * // In EvidentPatientService subtest files:
 * const context = useTestContext<IEvidentTestContext>('EvidentPatientService')
 * ```
 */
// eslint-disable-next-line @typescript-eslint/no-unnecessary-type-parameters
export function useTestContext<TContext = unknown>(contextKey: string): TContext {
    const manager = TestContextManager.getInstance()
    return manager.getContext<TContext>(contextKey)
}

/**
 * 🔧 **Context Manager Factory**
 * 
 * Creates a typed context manager instance for a specific test service.
 * This provides additional type safety and convenience methods.
 * 
 * @param contextKey - Unique identifier for the test service
 * @returns Typed context manager with convenience methods
 */
export function createTestContextManager<TContext>(contextKey: string): {
    setContext: (context: TContext) => void
    getContext: () => TContext
    clearContext: () => void
    hasContext: () => boolean
    contextKey: string
} {
    const manager = TestContextManager.getInstance()
    
    return {
        /**
         * Set context for this specific test service
         */
        setContext: (context: TContext): void => { manager.setContext(contextKey, context) },
        
        /**
         * Get context for this specific test service
         */
        getContext: (): TContext => manager.getContext<TContext>(contextKey),
        
        /**
         * Clear context for this specific test service
         */
        clearContext: (): void => { manager.clearContext(contextKey) },
        
        /**
         * Check if context exists for this specific test service
         */
        hasContext: (): boolean => manager.hasContext(contextKey),
        
        /**
         * Context key for this test service
         */
        contextKey
    }
}

/**
 * 🏷️ **Context Key Type Definition**
 * 
 * Type definition for context keys to ensure type safety.
 * Each project/service should define their own context keys.
 */
export type TestContextKey = string 
```

---

## 🎯 2. Parent Test File Template (Enterprise Architecture - **BEVORZUGT**)

**Pfad:** `test/unit/main/services/{ServiceName}/{ServiceName}.test.ts`

```typescript
// ==== IMPORTS ====
import path from 'path'
import { 
    describe, beforeEach, afterEach, vi
} from 'vitest'
import { YourServiceClass } from '@main/services/path/to/YourServiceClass.ts'
import { YourDependencyClass } from '@main/models/path/to/YourDependencyClass.ts'
// Import all your types, interfaces, models etc.

// Import Mock Factories
import {
    yourServiceMockFactory 
} from '@test/utils/mocks/services/path/to/YourServiceMockFactory.ts'
import { 
    yourDependencyMockFactory 
} from '@test/utils/mocks/models/path/to/YourDependencyMockFactory.ts'

// Import Fixture Factories (if applicable)
import {
    fixtureFactory, 
    ETestBundle
} from '@test/utils/fixtures/factory/index.ts'

// Import Test Environment Setup (if applicable)
import { 
    createQuickTestEnvironment, 
    type IYourTestEnvironment
} from '@test/utils/test-environment/setup/index.ts'

// Import Module Functions - Now WITHOUT function parameters!
import { createTestContextManager } from '@test/utils/TestContextManager.ts'

import { runModuleATests } from './YourServiceClass/moduleA.ts'
import { runModuleBTests } from './YourServiceClass/moduleB.ts'
import { runModuleCTests } from './YourServiceClass/moduleC.ts'

// 🎯 **Define context keys for this specific service (project-specific)**
export const CONTEXT_KEYS = {
    yourServiceName: 'YourServiceName' // Replace with actual service name
} as const

// ==== MOCKS ====
vi.mock('@main/services/path/to/YourDependencyService.ts', async() => {
    const mockedModule = await import('@test/utils/mocks/services/path/to/YourDependencyServiceMockFactory.ts')
    return mockedModule.mockYourDependencyServiceModule()
})

vi.mock('@main/services/path/to/YourOtherDependency.ts', async() => {
    const mockedModule = await import('@test/utils/mocks/services/path/to/YourOtherDependencyMockFactory.ts')
    return mockedModule.mockYourOtherDependencyModule()
})

vi.mock('@main/models/path/to/YourModel.ts', async() => {
    type YourModelType = typeof import('@main/models/path/to/YourModel.ts')
    const actual = await vi.importActual<YourModelType>('@main/models/path/to/YourModel.ts')

    const mockedYourModel = vi.fn().mockImplementation((
        ...args: readonly [...ConstructorParameters<typeof actual.YourModel>]
    ): IYourModel => {
        return new actual.YourModel(...args)
    })

    // Wichtig: Prototype beibehalten für Spying!
    mockedYourModel.prototype = actual.YourModel.prototype

    return {
        ...actual,
        // eslint-disable-next-line @typescript-eslint/naming-convention
        YourModel: mockedYourModel
    }
})

// Export ITestContext Interface
export interface ITestContext {
    service: InstanceType<typeof YourServiceClass>
    testEnv: IYourTestEnvironment
    testEnvironments: IYourTestEnvironment[]
    
    // Mock Factories
    yourServiceMockFactory: typeof yourServiceMockFactory
    yourDependencyMockFactory: typeof yourDependencyMockFactory
    mockYourModel: ReturnType<typeof vi.mocked<typeof YourModel>>
    
    // Fixture Bundles (if applicable)
    bundleA: IYourBundleAType
    bundleB: IYourBundleBType
    
    // Individual Test Fixtures
    testFixtureA: IYourFixtureAType
    testFixtureB: IYourFixtureBType
    testDependencyMock: IYourDependencyType
    
    // Computed Test Data
    computedTestDataA: IComputedDataType
    computedTestDataB: IComputedDataType
}

// ==== TESTS ====
describe('🏥 YourServiceClass - Enterprise Architecture', async() => {
    const mockYourModel = vi.mocked(YourModel)

    // 🎯 **Create typed context manager for this service**
    const contextManager = createTestContextManager<ITestContext>(
        CONTEXT_KEYS.yourServiceName
    )

    /* ==========================================
       ==== FIXTURE BUNDLE INITIALIZATION ====
       ==========================================
    */

    // Load your fixtures/bundles here (if needed)
    const [bundleA, bundleB] = await Promise.all([
        fixtureFactory.getBundle(ETestBundle.yourBundleA),
        fixtureFactory.getBundle(ETestBundle.yourBundleB, {
            overrides: {
                specificProperty: { yourOverride: true },
                anotherProperty: { readAll: true }
            }
        })
    ])
    
    beforeEach(async() => {
        // Setup isolierte Test-Umgebung
        const testEnv = await createQuickTestEnvironment()
        
        // Setup file paths on the mock instance (if applicable)
        const basePath = testEnv.tempDir
        vi.spyOn(yourServiceMockFactory.mockYourServiceInstance, 'yourPathProperty', 'get')
            .mockReturnValue(path.join(basePath, 'YOUR_FILE.EXT'))
        vi.spyOn(yourServiceMockFactory.mockYourServiceInstance, 'anotherPathProperty', 'get')
            .mockReturnValue(path.join(basePath, 'ANOTHER_FILE.EXT'))
            
        // Create service instance with mock dependencies
        const service = new YourServiceClass(yourServiceMockFactory.mockYourServiceInstance)

        // 🎯 **Set context via TestContextProvider instead of local variable**
        const context: ITestContext = {
            service,
            testEnv,
            testEnvironments: [],
            
            // Mock Factories
            yourServiceMockFactory,
            yourDependencyMockFactory,
            mockYourModel,
            
            // Fixture Bundles
            bundleA,
            bundleB,
            
            // Individual Test Fixtures (extracted from bundles)
            testFixtureA: bundleA.fixtureA,
            testFixtureB: bundleA.fixtureB,
            testDependencyMock: bundleB.dependencyMock,
            
            // Computed Test Data (derived from fixtures)
            computedTestDataA: bundleA.computedDataA[0],
            computedTestDataB: bundleB.computedDataB.find(
                (item: IComputedDataType) => item.id === bundleA.fixtureA.referenceId
            )
        }

        // 🚀 **Set context via central manager**
        contextManager.setContext(context)
    })

    afterEach(async() => {
        // Get current context for cleanup
        const context = contextManager.getContext()
        
        await context.testEnv.cleanup()
        
        // Cleanup aller zusätzlichen Test-Umgebungen
        for (const env of context.testEnvironments) {
            await env.cleanup()
        }
        
        // 🧹 **Clear context after test**
        contextManager.clearContext()
    })

    // 🎯 **Execute all test modules**
    runModuleATests()
    runModuleBTests()
    runModuleCTests()
})
```

---

## 🎯 3. Sub Test File Template (Enterprise Architecture - **BEVORZUGT**)

**Pfad:** `test/unit/main/services/{ServiceName}/{ServiceName}/moduleA.ts`

⚠️ **WICHTIG:** Sub-Test-Dateien dürfen **KEINEN** "test"-Prefix haben (z.B. **NICHT** `moduleA.test.ts`), sonst werden sie als separate Test-Dateien erkannt und schlagen fehl, da sie nur Funktionen exportieren!

```typescript
// ==== IMPORTS ====
import { describe, beforeEach, it, expect, vi, type MockInstance } from 'vitest'
import { useTestContext } from '@test/utils/TestContextManager.ts'
import { CONTEXT_KEYS, type ITestContext } from '../YourServiceClass.test.ts'

export const runModuleATests = (): void => {
    describe('🔧 Module A Functionality', () => {
        let context: ITestContext

        beforeEach(() => {
            // 🎯 **Use project-specific context key from parent test file**
            context = useTestContext<ITestContext>(CONTEXT_KEYS.yourServiceName)
        })

        describe('methodA()', () => {
            describe('Positive Tests ✅', () => {
                it('should perform operation successfully', async() => {
                    // Arrange
                    const input = 'test-input'
                    const expectedOutput = 'expected-output'

                    // Setup mocks using context
                    context.yourDependencyMockFactory.mockMethodA.mockResolvedValue(expectedOutput)

                    // Act
                    const result = await context.service.methodA(input)

                    // Assert
                    expect(result).toBe(expectedOutput)
                    expect(
                        context.yourDependencyMockFactory.mockMethodA
                    ).toHaveBeenCalledExactlyOnceWith(input)
                })

                it('should process complex data correctly', async() => {
                    // Arrange
                    const expectedResult = {
                        id: context.testFixtureA.id,
                        name: context.testFixtureA.name,
                        processedData: context.computedTestDataA
                    }

                    // Act
                    const result = await context.service.methodA(context.testFixtureA)

                    // Assert
                    expect(result).toEqual(expectedResult)
                })
            })

            describe('Negative Tests ❌', () => {
                it('should handle error cases properly', async() => {
                    // Arrange
                    const input = 'invalid-input'
                    const expectedError = new Error('Invalid input')

                    // Setup mocks using context
                    context.yourDependencyMockFactory.mockMethodA.mockRejectedValue(expectedError)

                    // Act & Assert
                    await expect(context.service.methodA(input)).rejects.toThrow('Invalid input')
                })
            })
        })

        describe('methodB()', () => {
            describe('Positive Tests ✅', () => {
                let spyOnInternalMethod: MockInstance

                beforeEach(() => {
                    spyOnInternalMethod = vi.spyOn(
                        context.service, 'internalHelperMethod'
                    ).mockResolvedValue(context.computedTestDataB)
                })

                it('should call internal methods and return processed data', async() => {
                    // Act
                    const result = await context.service.methodB()

                    // Assert
                    expect(result).toEqual({
                        id: context.computedTestDataB.id,
                        processed: true
                    })
                    expect(spyOnInternalMethod).toHaveBeenCalledExactlyOnceWith()
                })

                it('should handle undefined case when internal method returns null', async() => {
                    // Arrange
                    spyOnInternalMethod.mockResolvedValue(null)

                    // Act
                    const result = await context.service.methodB()

                    // Assert
                    expect(result).toBeUndefined()
                    expect(spyOnInternalMethod).toHaveBeenCalledExactlyOnceWith()
                })
            })
        })
    })
}
```

---

## 📋 AI Agent Implementation Rules

### 🔧 Required Steps for Implementation (Enterprise Architecture)

1. **Create TestContextManager** (if not exists)
   - Copy 1:1 the TestContextManager code above
   - Place in `test/utils/TestContextManager.ts`

2. **Create Parent Test File**
   - Use Parent Test File Template
   - Replace `YourServiceClass` with actual service name
   - Replace `yourServiceName` in CONTEXT_KEYS with actual service name
   - **Implement vi.mock() statements** with proper mock factory imports
   - **Import all mock factories** needed for the service
   - Define complete `ITestContext` interface with all needed properties
   - Setup proper imports, mocks, and fixtures in `beforeEach`

3. **Create Sub Test Files**
   - Use Sub Test File Template for each test module
   - **WICHTIG:** Dateiname **OHNE** "test"-Prefix (z.B. `moduleA.ts`, **NICHT** `moduleA.test.ts`)
   - Import `CONTEXT_KEYS` and `ITestContext` from parent test file
   - Use `useTestContext<ITestContext>(CONTEXT_KEYS.yourServiceName)` in `beforeEach`
   - Access all test data via `context.property`

### 🎯 Naming Conventions

- **Parent Test File:** `{ServiceName}.test.ts`
- **Sub Test Files:** `{ServiceName}/{moduleName}.ts` ⚠️ **OHNE "test"-Prefix!**
- **Context Key:** Use PascalCase service name (e.g., `DampsoftPatientService`)
- **Export Function:** `run{ModuleName}Tests()`
- **Mock Factories:** Import and include in context as `{serviceName}MockFactory`

### ✅ Quality Checklist

- [ ] TestContextManager is identical to reference implementation
- [ ] Parent test file includes all vi.mock() statements with mock factories
- [ ] Parent test file imports all necessary mock factories
- [ ] Parent test file defines complete `ITestContext` interface
- [ ] All sub test files import `CONTEXT_KEYS` from parent
- [ ] All sub test files use `useTestContext<ITestContext>(CONTEXT_KEYS.serviceName)`
- [ ] Sub test files have **NO** "test"-Prefix in filename
- [ ] No function parameters passed between test files
- [ ] Proper TypeScript typing throughout
- [ ] Consistent error handling with descriptive messages

---

## 🚀 Alternative Solution (Simplified - **NICHT BEVORZUGT**)

⚠️ **WICHTIGER HINWEIS:** Diese Alternative sollte **NUR** verwendet werden, wenn:
- Keine `beforeEach`-Definitionen nötig sind, die nach dem Hoisting verfügbar sein müssen
- Sehr einfache Test-Szenarien vorliegen
- **Die Enterprise-Architektur ist IMMER die bevorzugte Lösung!**

### Alternative Parent Test File

```typescript
// ==== IMPORTS ====
import { describe } from 'vitest'
import { YourServiceClass } from '@main/services/path/to/YourServiceClass.ts'

// Direkte Imports der Sub-Test-Module (ohne Context Manager)
import { runModuleATests } from './YourServiceClass/moduleA.ts'
import { runModuleBTests } from './YourServiceClass/moduleB.ts'
import { runModuleCTests } from './YourServiceClass/moduleC.ts'

// Global verfügbare Test-Daten (auf Hoisted-Ebene)
export const globalTestData = {
    service: new YourServiceClass(),
    testFixtureA: { id: 1, name: 'Test' },
    testFixtureB: { id: 2, name: 'Another Test' }
}

// ==== TESTS ====
describe('🏥 YourServiceClass - Simple Architecture (NICHT BEVORZUGT)', () => {
    // 🎯 **Direkte Ausführung der Test-Module**
    runModuleATests()
    runModuleBTests()
    runModuleCTests()
})
```

### Alternative Sub Test File

```typescript
// ==== IMPORTS ====
import { describe, it, expect } from 'vitest'
import { globalTestData } from '../YourServiceClass.test.ts'

export const runModuleATests = (): void => {
    describe('🔧 Module A Functionality', () => {
        describe('methodA()', () => {
            it('should work with global data', async() => {
                // Direct access to global test data
                const result = await globalTestData.service.methodA(globalTestData.testFixtureA)
                
                expect(result).toBeDefined()
            })
        })
    })
}
```

### Warum die Alternative NICHT bevorzugt ist:

❌ **Keine Isolation** zwischen Tests  
❌ **Schwer skalierbar** bei komplexen Services  
❌ **Keine dynamische Kontext-Erstellung** in `beforeEach`  
❌ **Mock-Management** wird kompliziert  
❌ **Shared State** Probleme bei parallelen Tests  
❌ **Schlechte Wartbarkeit** bei großen Test-Suites  

---

## 🏆 Warum Enterprise Architecture Bevorzugt Ist

### 🚀 Benefits der Enterprise Architecture

✅ **100% Portable** - Works across any TypeScript project  
✅ **Type-Safe** - Full TypeScript generics support  
✅ **Scalable** - Registry pattern for multiple services  
✅ **Clean Architecture** - Clear separation of concerns  
✅ **Enterprise Ready** - Standardized patterns  
✅ **Zero Boilerplate** - No function parameters between modules  
✅ **Perfect Isolation** - Each test gets fresh context  
✅ **Mock Management** - Centralized mock factory handling  
✅ **Dynamic Setup** - `beforeEach` context creation support  
✅ **Parallel Safe** - No shared state issues  

### ⭐ Fazit

**IMMER** die Enterprise Architecture mit TestContextManager verwenden, außer in sehr simplen Fällen wo garantiert keine `beforeEach`-Logik benötigt wird. Die Alternative ist nur als Notlösung gedacht.
