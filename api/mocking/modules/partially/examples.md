
# Partially Mocking

<details><summary>Click to expand..</summary>

<br><br>

###  vi.importActual()

<details><summary>Click to expand..</summary>


## ❗ Kritisische Anweisungen

*   Verwende `vi.mock('package-name')`, um ein NPM-Package zu mocken.
*   Nutze `vi.importActual` innerhalb der Mock-Factory, um Teile des Originalmoduls beizubehalten.
*   Definiere die benötigten Funktionen oder Klassen mit `vi.fn()` oder simuliere das erwartete Verhalten.
*   Stelle sicher, dass die gemockten Funktionen das erwartete Verhalten für deine Tests simulieren.
*   Verwende `vi.mocked` für typisierte Mock-Referenzen.

## ✅ Beispiele

### Mocking mit `vi.importActual`

```typescript
// test/my-module.test.ts
import * as somePackage from 'some-package'; // Ersetze 'some-package' durch den tatsächlichen Package-Namen
import { vi, describe, it, expect } from 'vitest';

// Mock das Package, behalte aber die tatsächlichen Implementierungen bei
vi.mock('some-package', async () => {
  const actualPackage = await vi.importActual<typeof somePackage>('some-package');
  return {
    ...actualPackage,
    // Überschreibe spezifische Funktionen mit Mocks
    someFunction: vi.fn(),
    anotherFunction: vi.fn(),
  };
});

// Typisierte Referenz zum gemockten Modul
const mockedSomePackage = vi.mocked(somePackage, true);

describe('Package Operationen', () => {
  it('sollte someFunction mocken', async () => {
    // Setze die Implementierung für die gemockte Funktion
    mockedSomePackage.someFunction.mockResolvedValue('mocked result');

    // Rufe die Funktion auf, die someFunction verwendet
    const result = await somePackage.someFunction();

    // Überprüfe, ob die gemockte Funktion aufgerufen wurde
    expect(mockedSomePackage.someFunction).toHaveBeenCalled();
    expect(result).toBe('mocked result');
  });

  it('sollte andere Funktionen des Packages weiterhin nutzen', async () => {
    // Wenn 'anotherFunction' nicht gemockt wurde, wird die tatsächliche Implementierung verwendet
    // Hier könntest du testen, ob die tatsächliche Funktion korrekt aufgerufen wird oder das erwartete Ergebnis liefert
    // expect(mockedSomePackage.anotherFunction).toHaveBeenCalled(); // Nur wenn gemockt
    // expect(somePackage.originalFunction()).toBe('expected result'); // Testet eine nicht gemockte Funktion
  });
});
```

</details>

<br><br>


</details>

