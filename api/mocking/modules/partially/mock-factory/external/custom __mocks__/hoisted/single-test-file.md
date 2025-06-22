## Vereinfachtes Beispiel: Zentrale Mock-Datei für einen einzigen Service

Dieses Beispiel reduziert die Architektur auf das absolute Minimum. Es zeigt, wie man eine **einzige Methode** eines einfachen Services mit einer zentralen, gehoisteten Mock-Datei mockt. Der Fokus liegt auf der **Klarheit der Architektur** und dem korrekten Import-Verhalten.

### 🎯 Überblick der Architektur

Wir verwenden eine externe Mock-Datei mit `vi.hoisted()`, um einen Mock vorzubereiten. Dieser Mock wird dann in der Testdatei durch einen speziellen Import aktiviert, um eine einzelne Service-Methode zu ersetzen.

---

### 🗂️ Dateistruktur

```
test/
├── utils/mocks/SimpleCalculator/
│   └── SimpleCalculator.ts       # 🔑 ZENTRALE MOCK-DATEI
└── unit/main/services/
    └── SimpleCalculator.test.ts  # 🏠 TESTDATEI
```
*(Annahme: Der echte Service liegt unter `src/main/services/SimpleCalculator.ts`)*

---

### 🔑 1. Zentrale Mock-Datei (`test/utils/mocks/SimpleCalculator/SimpleCalculator.ts`)

Diese Datei enthält die gesamte Mock-Logik für unseren `SimpleCalculator`.

<example>
```typescript
// ==== DEPENDENCIES ====
import { vi } from 'vitest'

// ==== 1. HOISTED MOCK SETUP ====
// Definiere den Mock für die 'add' Methode in einem `vi.hoisted` Block.
// Dies stellt sicher, dass der Mock existiert, bevor `vi.mock` ausgeführt wird.
const mockProvider = vi.hoisted(() => {
    const add = vi.fn()
    return { add }
})

// ==== 2. MODULE MOCK ====
// Mocke den Pfad zum echten Service.
// Die Factory-Funktion (`() => ({...})`) ersetzt die Exporte des Moduls.
vi.mock('@main/services/SimpleCalculator.ts', () => ({
    // Ersetze die echte 'add' Methode durch unseren gehoisteten Mock.
    add: mockProvider.add
}))

// ==== 3. MOCK ACCESS (VEREINFACHT) ====
// Ein einfacher Export, um typsicheren Zugriff auf den Mock in Tests zu ermöglichen.
export const calculatorMocks = {
    add: mockProvider.add
}
```
</example>

---

### 🏠 2. Testdatei (`test/unit/main/services/SimpleCalculator.test.ts`)

Diese Datei importiert die Mock-Konfiguration und testet, ob die `add`-Methode durch den Mock korrekt aufgerufen wird.

<example>
```typescript
// ==== DEPENDENCIES ====
import { describe, it, expect } from 'vitest'

// Importiere die Methode, die wir testen wollen.
// Vitest wird diesen Import auf unseren Mock umleiten.
import { add } from '@main/services/SimpleCalculator.ts'

// 🔥 KRITISCHER IMPORT: Diese Zeile aktiviert die Mock-Konfiguration.
// Sie MUSS importiert werden, damit `vi.mock()` aus der zentralen Datei greift.
// KEIN "from" verwenden, um den Hoisting-Mechanismus zu aktivieren!
import '@test/utils/mocks/SimpleCalculator/SimpleCalculator.ts'

// Regulärer Import, um auf die Mock-Instanz zuzugreifen und Assertions zu machen.
import { calculatorMocks } from '@test/utils/mocks/SimpleCalculator/SimpleCalculator.ts'

describe('SimpleCalculator', () => {
    it('sollte die gemockte add-Methode aufrufen', () => {
        // Arrange
        const a = 5
        const b = 10

        // Act
        // Rufe die importierte 'add'-Methode auf.
        // Dank des Mocks wird nun unsere vi.fn()-Instanz aufgerufen.
        add(a, b)

        // Assert
        // Überprüfe, ob unser Mock korrekt aufgerufen wurde.
        expect(calculatorMocks.add).toHaveBeenCalledTimes(1)
        expect(calculatorMocks.add).toHaveBeenCalledWith(a, b)
    })
})
```
</example>

### ⚡ **WICHTIGER HINWEIS zum Import:**
Der Mechanismus bleibt identisch: Der Import ohne `from` ist entscheidend, um die Mocks korrekt zu "hoisten", bevor der Testcode ausgeführt wird.

```typescript
// ✅ RICHTIG: Lädt und aktiviert die gehoisteten Mocks
import '@test/utils/mocks/SimpleCalculator/SimpleCalculator.ts'

// ❌ FALSCH: Die Mocks werden NICHT aktiviert
import { calculatorMocks } from '@test/utils/mocks/SimpleCalculator/SimpleCalculator.ts'
```

--- 