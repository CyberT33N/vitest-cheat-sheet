# Vitest: Using `vi.hoisted` with Async Import for `vi.mock` Factories

## Problem Statement

When using `vi.mock` with a factory function, if that factory attempts to use a variable imported from another module (e.g., a mock class), you might encounter a `ReferenceError` like "Cannot access '...' before initialization." This happens because `vi.mock` calls are hoisted to the top of the file and their factories are executed before regular module imports are fully resolved.

```typescript
// ❌ INCORRECT - This can lead to a hoisting ReferenceError
import { MyMockService } from './mocks/my-mock-service';

vi.mock('./src/original-service', () => {
  // MyMockService might not be initialized here yet
  return { OriginalService: MyMockService };
});
```

## ❗ Critical Rules

*   **ALWAYS** use `vi.hoisted` to load modules or variables that are needed inside a `vi.mock` factory if they are imported from external files.
*   If the mock module needs to be imported (e.g., a mock class from a separate file), **MUST** use a dynamic `await import(...)` inside the `vi.hoisted` factory.
*   The `vi.hoisted` factory function **MUST** return an object containing the loaded mock(s).
*   The `vi.mock` factory function **MUST** be declared `async` if it needs to `await` the promise returned by `vi.hoisted` (when the `vi.hoisted` factory itself is `async`).
*   **MUST** `await` the promise returned by `vi.hoisted` within the `async vi.mock` factory before accessing the mock.
*   Ensure variable names used for hoisted mocks and in the `vi.mock` factory comply with project linting rules (e.g., `camelCase` for variables holding class constructors if required by linters).

## ✅ Solution: `vi.hoisted` with Async Import and Async Mock Factory

This pattern ensures that the mock implementation is loaded and available *before* the `vi.mock` factory for the target module attempts to use it.

<example>
  // ✅ CORRECT - Using vi.hoisted with async import and async vi.mock factory

  // File: my-service.test.ts
  import { describe, it, expect, vi } from 'vitest';
  import { OriginalService } from './src/original-service'; // The service to be tested

  // 1. Use vi.hoisted with an async factory to dynamically import the mock
  const mockServiceProvider = vi.hoisted(async () => {
    const mockModule = await import('./mocks/my-mock-service.ts');
    // Return an object, using a camelCase key if required by linters
    return { mockOriginalService: mockModule.MockOriginalService };
  });

  // 2. Make the vi.mock factory async and await the promise from vi.hoisted
  vi.mock('./src/original-service', async () => {
    const { mockOriginalService } = await mockServiceProvider; // Destructure the actual mock class
    return {
      // eslint-disable-next-line @typescript-eslint/naming-convention
      OriginalService: mockOriginalService // Use the resolved mock class
    };
  });

  describe('OriginalService', () => {
    it('should use the mocked implementation', () => {
      const serviceInstance = new OriginalService();
      // Assuming MockOriginalService has a specific method or behavior
      expect(vi.isMockFunction(serviceInstance.someMethod)).toBe(true);
      // ... further assertions ...
    });
  });

  // File: ./mocks/my-mock-service.ts
  // export class MockOriginalService {
  //   constructor() {
  //     // Mock constructor logic if needed
  //     vi.fn()(this, 'constructor');
  //   }
  //   someMethod = vi.fn(() => 'mocked value');
  //   // ... other mocked methods ...
  // }
</example>

## 🤔 Rationale / Why this pattern works

1.  **`vi.hoisted(async () => ...)`**:
    *   The code inside `vi.hoisted` is executed *before* other static imports and `vi.mock` calls are fully processed.
    *   Using `async () => { await import(...) }` allows for the dynamic and asynchronous loading of your mock module (e.g., `MockElectronBackendService.ts`).
    *   It returns a `Promise` that resolves to an object containing your mock (e.g., `{ mockOriginalService: MockOriginalServiceClass }`).

2.  **`vi.mock('module-to-mock', async () => ...)`**:
    *   The factory function for `vi.mock` can be `async`.
    *   Inside this `async` factory, you `await` the promise returned by `vi.hoisted`. This ensures that the `MockOriginalServiceClass` (or whatever you named it) is fully loaded and available.
    *   You then return the standard mock structure, e.g., `{ OriginalService: resolvedMockClass }`.

This two-step asynchronous process correctly handles the timing and availability of the mock implementation, resolving the hoisting-related `ReferenceError`.

## 🔗 Related Vitest APIs

*   [`vi.mock()`](https://vitest.dev/api/vi.html#vi-mock)
*   [`vi.hoisted()`](https://vitest.dev/api/vi.html#vi-hoisted)
*   Dynamic `import()`

By following this pattern, you can reliably use externally defined mock classes within your `vi.mock` factories, promoting cleaner and more maintainable test setups, especially when mock implementations are complex or reused. 