
## Zentrale Mockdatei mit MockFactory (für mehrfache Nutzung)

Diese Architektur ist **bevorzugt**, wenn ein Mock von mehreren Testdateien benötigt wird. Sie nutzt eine zentrale Mockdatei mit `vi.hoisted()` und einer MockFactory. Das **Beispiel** kann **NATÜRLICH** auch für eine **einzelne Testdatei** angewendet werden.

### 🎯 Überblick der Architektur

Diese Architektur löst **Hoisting-Probleme** bei Vitest-Mocks durch eine **externe Mock-Datei** mit dem `vi.hoisted()` Pattern und einen **kritischen Import** in der Haupttestdatei.

---

### 🗂️ Dateistruktur

```
test/
├── utils/mocks/ServiceName/
│   └── ServiceName.ts                    # 🔑 ZENTRALE MOCK-DATEI
├── unit/main/services/
│   ├── ServiceName.test.ts               # 🏠 HAUPTTESTDATEI
│   └── ServiceName-modules/              # 📁 MODULARE TESTDATEIEN
│       ├── feature1.ts
│       ├── feature2.ts
│       └── feature3.ts
```

---

### 🔑 1. Zentrale Mock-Datei (`test/utils/mocks/ServiceName/ServiceName.ts`)

<example>
```typescript
// ==== DEPENDENCIES ====
import { Mock, vi } from 'vitest'
import { ElectronMessage, ElectronMessageType } from '@main/models/ElectronMessage.ts'

// ==== HOISTED MOCK SETUP ====
const mockProvider = vi.hoisted(() => {
    // Alle Mock-Funktionen definieren
    const handleFeature1 = vi.fn(async() =>
        Promise.resolve(new ElectronMessage(ElectronMessageType.feature1))
    )
    const handleFeature2 = vi.fn(async() =>
        Promise.resolve(new ElectronMessage(ElectronMessageType.feature2))
    )
    const handleFeature3 = vi.fn(async() =>
        Promise.resolve(new ElectronMessage(ElectronMessageType.feature3))
    )

    return {
        handleFeature1,
        handleFeature2,
        handleFeature3
    }
})

// ==== MODULE MOCKS (HOISTED) ====
vi.mock('@main/services/message-handlers/Feature1Handler.ts', () => ({
    handleFeature1: mockProvider.handleFeature1
}))

vi.mock('@main/services/message-handlers/Feature2Handler.ts', () => ({
    handleFeature2: mockProvider.handleFeature2
}))

vi.mock('@main/services/message-handlers/Feature3Handler.ts', () => ({
    handleFeature3: mockProvider.handleFeature3
}))

// ==== MOCK FACTORY ====
class ServiceMockFactory {
    private static _instance: ServiceMockFactory | undefined

    // Direct mock properties
    public readonly handleFeature1 = mockProvider.handleFeature1
    public readonly handleFeature2 = mockProvider.handleFeature2
    public readonly handleFeature3 = mockProvider.handleFeature3

    public static getInstance(): ServiceMockFactory {
        ServiceMockFactory._instance ??= new ServiceMockFactory()
        return ServiceMockFactory._instance
    }

    public getFeature1Mocks(): { handleFeature1: Mock } {
        return { handleFeature1: mockProvider.handleFeature1 }
    }

    public getFeature2Mocks(): { handleFeature2: Mock } {
        return { handleFeature2: mockProvider.handleFeature2 }
    }
}

// ==== CONVENIENCE EXPORT ====
export const serviceMocks = ServiceMockFactory.getInstance()
```
</example>

---

### 🏠 2. Haupttestdatei (`test/unit/main/services/ServiceName.test.ts`)

<example>
```typescript
// ==== DEPENDENCIES ====
import { describe } from 'vitest'

// 🔥 KRITISCHER IMPORT: Mock-Datei MUSS hier einmalig importiert werden
// ⚠️  KEIN "from" verwenden - nur import um hoisted mocks zu laden!
import '@test/utils/mocks/ServiceName/ServiceName.ts'

// Import modularized test functions
import { runFeature1Tests } from './ServiceName-modules/feature1.ts'
import { runFeature2Tests } from './ServiceName-modules/feature2.ts'
import { runFeature3Tests } from './ServiceName-modules/feature3.ts'

describe('[ServiceName.ts] - src/main/services/ServiceName.ts', () => {
    describe('handleMessages()', () => {
        // Execute all modularized test suites
        runFeature1Tests()
        runFeature2Tests()
        runFeature3Tests()
    })
})
```
</example>

### ⚡ **WICHTIGER HINWEIS zum Import:**
```typescript
// ✅ RICHTIG: Lädt hoisted mocks
import '@test/utils/mocks/ServiceName/ServiceName.ts'

// ❌ FALSCH: Hoisted mocks werden NICHT geladen
import { serviceMocks } from '@test/utils/mocks/ServiceName/ServiceName.ts'
```

---

### 📁 3. Modulare Testdatei (`test/unit/main/services/ServiceName-modules/feature1.ts`)

<example>
```typescript
// ==== DEPENDENCIES ====
import { beforeEach, describe, expect, it, Mock } from 'vitest'
import { ElectronMessage, ElectronMessageType } from '@main/models/ElectronMessage.ts'
import { ServiceName } from '@main/services/ServiceName.ts'
import { serviceMocks } from '@test/utils/mocks/ServiceName/ServiceName.ts'

export function runFeature1Tests(): void {
    describe('Feature1', () => {
        let serviceInstance: ServiceName
        let feature1Mocks: {
            handleFeature1: Mock
        }

        beforeEach(() => {
            serviceInstance = new ServiceName()
            // Lokale typisierte Mock-Gruppe für bessere Organisation
            feature1Mocks = serviceMocks.getFeature1Mocks()
        })

        describe('[✅ Success Cases]', () => {
            it('sollte Nachrichten korrekt an den handleFeature1 Handler weiterleiten', async() => {
                // Arrange
                const inputMsg = new ElectronMessage(ElectronMessageType.feature1)
                inputMsg.data = { someProperty: 'testValue' }

                // Act
                await serviceInstance.handleMessages(inputMsg)

                // Assert
                expect(feature1Mocks.handleFeature1).toHaveBeenCalledTimes(1)
                expect(feature1Mocks.handleFeature1)
                    .toHaveBeenCalledWith(inputMsg, expect.anything())
            })
        })

        describe('[❌ Error Cases]', () => {
            it('sollte Fehler korrekt behandeln', async() => {
                // Arrange
                const inputMsg = new ElectronMessage(ElectronMessageType.feature1)
                feature1Mocks.handleFeature1.mockRejectedValueOnce(new Error('Test Error'))

                // Act & Assert
                await expect(serviceInstance.handleMessages(inputMsg))
                    .rejects.toThrow('Test Error')

                expect(feature1Mocks.handleFeature1).toHaveBeenCalledTimes(1)
            })
        })
    })
}
```
</example>
