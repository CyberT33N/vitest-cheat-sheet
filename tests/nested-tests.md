# 🏗️ Enterprise Test Context Architecture - Reference Guide

## 📋 Architektur-Übersicht

Diese Regel definiert eine **zentrale, typisierte Test-Context-Management-Architektur** für Enterprise-TypeScript-Projekte. Die Architektur eliminiert Boilerplate-Code und ermöglicht saubere Trennung von Test-Modulen.

### Komponenten
1. **TestContextManager** - Zentrale Singleton-Registry für Test-Kontexte
2. **Parent Test File** - Haupttest-Datei mit Kontext-Setup
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

## 🎯 2. Parent Test File Template

**Pfad:** `test/unit/main/services/{ServiceName}/{ServiceName}.test.ts`

```typescript
// ==== IMPORTS ====
import { 
    describe, beforeEach, afterEach, vi
} from 'vitest'
import { YourServiceClass } from '@main/services/path/to/YourServiceClass.ts'
// Import your dependencies, types, fixtures etc.

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
// Your vi.mock() statements here...

// Export ITestContext Interface
export interface ITestContext {
    service: InstanceType<typeof YourServiceClass>
    // Add all your test context properties here
    mockDependencyA: SomeType
    mockDependencyB: SomeType
    testFixtureA: SomeType
    testFixtureB: SomeType
    // ... more properties as needed
}

// ==== TESTS ====
describe('🏥 YourServiceClass - Enterprise Architecture', async() => {
    // 🎯 **Create typed context manager for this service**
    const contextManager = createTestContextManager<ITestContext>(
        CONTEXT_KEYS.yourServiceName
    )

    /* ==========================================
       ==== FIXTURE BUNDLE INITIALIZATION ====
       ==========================================
    */

    // Load your fixtures/bundles here (if needed)
    // const [bundleA, bundleB] = await Promise.all([...])
    
    beforeEach(async() => {
        // Setup your test environment, mocks, fixtures etc.
        
        // Create service instance
        const service = new YourServiceClass(/* dependencies */)

        // 🎯 **Set context via TestContextProvider instead of local variable**
        const context: ITestContext = {
            service,
            // Set all your context properties here
            mockDependencyA: /* your mock */,
            mockDependencyB: /* your mock */,
            testFixtureA: /* your fixture */,
            testFixtureB: /* your fixture */,
            // ... more properties
        }

        // 🚀 **Set context via central manager**
        contextManager.setContext(context)
    })

    afterEach(async() => {
        // Get current context for cleanup (if needed)
        const context = contextManager.getContext()
        
        // Perform any cleanup operations
        // await context.someCleanupOperation()
        
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

## 🎯 3. Sub Test File Template

**Pfad:** `test/unit/main/services/{ServiceName}/{ServiceName}/moduleA.ts`

```typescript
// ==== IMPORTS ====
import { describe, beforeEach, it, expect, vi } from 'vitest'
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
                    context.mockDependencyA.someMethod.mockResolvedValue(expectedOutput)

                    // Act
                    const result = await context.service.methodA(input)

                    // Assert
                    expect(result).toBe(expectedOutput)
                    expect(context.mockDependencyA.someMethod).toHaveBeenCalledExactlyOnceWith(input)
                })
            })

            describe('Negative Tests ❌', () => {
                it('should handle error cases properly', async() => {
                    // Arrange
                    const input = 'invalid-input'
                    const expectedError = new Error('Invalid input')

                    // Setup mocks using context
                    context.mockDependencyA.someMethod.mockRejectedValue(expectedError)

                    // Act & Assert
                    await expect(context.service.methodA(input)).rejects.toThrow('Invalid input')
                })
            })
        })

        describe('methodB()', () => {
            describe('Positive Tests ✅', () => {
                it('should process data correctly', async() => {
                    // Use context.testFixtureA, context.service etc.
                    const result = await context.service.methodB(context.testFixtureA)
                    
                    expect(result).toBeDefined()
                    // More assertions...
                })
            })
        })
    })
}
```

---

## 📋 AI Agent Implementation Rules

### 🔧 Required Steps for Implementation

1. **Create TestContextManager** (if not exists)
   - Copy 1:1 the TestContextManager code above
   - Place in `test/utils/TestContextManager.ts`

2. **Create Parent Test File**
   - Use Parent Test File Template
   - Replace `YourServiceClass` with actual service name
   - Replace `yourServiceName` in CONTEXT_KEYS with actual service name
   - Define complete `ITestContext` interface with all needed properties
   - Setup proper imports, mocks, and fixtures in `beforeEach`

3. **Create Sub Test Files**
   - Use Sub Test File Template for each test module
   - Import `CONTEXT_KEYS` and `ITestContext` from parent test file
   - Use `useTestContext<ITestContext>(CONTEXT_KEYS.yourServiceName)` in `beforeEach`
   - Access all test data via `context.property`

### 🎯 Naming Conventions

- **Parent Test File:** `{ServiceName}.test.ts`
- **Sub Test Files:** `{ServiceName}/{moduleName}.ts`
- **Context Key:** Use PascalCase service name (e.g., `DampsoftPatientService`)
- **Export Function:** `run{ModuleName}Tests()`

### ✅ Quality Checklist

- [ ] TestContextManager is identical to reference implementation
- [ ] Parent test file defines complete `ITestContext` interface
- [ ] All sub test files import `CONTEXT_KEYS` from parent
- [ ] All sub test files use `useTestContext<ITestContext>(CONTEXT_KEYS.serviceName)`
- [ ] No function parameters passed between test files
- [ ] Proper TypeScript typing throughout
- [ ] Consistent error handling with descriptive messages

### 🚀 Benefits Achieved

✅ **100% Portable** - Works across any TypeScript project  
✅ **Type-Safe** - Full TypeScript generics support  
✅ **Scalable** - Registry pattern for multiple services  
✅ **Clean Architecture** - Clear separation of concerns  
✅ **Enterprise Ready** - Standardized patterns  
✅ **Zero Boilerplate** - No function parameters between modules
