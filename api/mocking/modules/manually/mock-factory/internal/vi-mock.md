

# vi.mock()


## Example: Direct Mocking (`fs/promises`)

```typescript
import * as fsPromises from 'fs/promises';
import { vi, describe, it, expect } from 'vitest';

vi.mock('fs/promises', () => ({
  mkdir: vi.fn(),
  writeFile: vi.fn(),
  readFile: vi.fn(),
  readdir: vi.fn(),
}));

const mockedFsPromises = vi.mocked(fsPromises, true); // deep mocking

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






<br><br>
<br><br>

## Example 2 - Directly usage of the mocked module

<details><summary>Click to expand..</summary>

<br><br>

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { userService } from '../src/userService.js'

// VITEST MOCK: Mit expliziter Factory-Funktion
vi.mock('axios', () => ({
  default: {
    get: vi.fn(),
    post: vi.fn(),
    delete: vi.fn()
  }
}))

// Import des gemockten axios NACH dem vi.mock
const axios = await import('axios')

describe('UserService - Vitest Mock (Factory Variante)', () => {
  beforeEach(() => {
    // Reset aller Mocks vor jedem Test
    vi.clearAllMocks()
  })

  it.only('sollte einen Benutzer abrufen und Aufruf prüfbar machen', async () => {
    // ✅ Setup des Mock-Rückgabewerts
    const mockUserData = { id: 1, name: 'Vitest User', email: 'vitest@test.com' }
    axios.default.get.mockResolvedValue({ data: mockUserData })
    
    const user = await userService.getUser(1)
    
    // ✅ Daten-Assertion
    expect(user).toEqual(mockUserData)
    
    // ✅ Mock-Aufruf-Assertions (DAS FUNKTIONIERT!)
    expect(axios.default.get).toHaveBeenCalledTimes(1)
    expect(axios.default.get).toHaveBeenCalledWith('https://api.example.com/users/1')
  })
})

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
