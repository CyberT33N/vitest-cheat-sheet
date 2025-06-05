*   **MUST:** Wenn mit **Sub-Regeln** gearbeitet wird, die als **Funktion** exportiert und in der **Haupttestdatei** importiert werden, dürfen die **Sub-Testdateien** **NIEMALS** mit dem Präfix **test.ts** benannt werden, weil sie sonst als **Test-Suite** erkannt werden und dann nicht geladen werden können.

# Option 1 - vi.hoisted() mit externer Mock-Datei (Preferred)

## 🎯 Überblick der Architektur

Diese Architektur löst **Hoisting-Probleme** bei Vitest-Mocks durch eine **externe Mock-Datei** mit dem `vi.hoisted()` Pattern und einen **kritischen Import** in der Haupttestdatei.

---

## 🗂️ Dateistruktur

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

## 🔑 1. Zentrale Mock-Datei (`test/utils/mocks/ServiceName/ServiceName.ts`)

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

---

## 🏠 2. Haupttestdatei (`test/unit/main/services/ServiceName.test.ts`)

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

### ⚡ **WICHTIGER HINWEIS zum Import:**
```typescript
// ✅ RICHTIG: Lädt hoisted mocks
import '@test/utils/mocks/ServiceName/ServiceName.ts'

// ❌ FALSCH: Hoisted mocks werden NICHT geladen
import { serviceMocks } from '@test/utils/mocks/ServiceName/ServiceName.ts'
```

---

## 📁 3. Modulare Testdatei (`test/unit/main/services/ServiceName-modules/feature1.ts`)

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

---

## 🚀 4. Warum diese Architektur?

### ✅ **Vorteile:**
- **Löst Hoisting-Probleme:** `vi.hoisted()` stellt sicher, dass Mocks vor der Modulauflösung verfügbar sind
- **Zentrale Mock-Verwaltung:** Alle Mocks an einem Ort definiert
- **Modulare Tests:** Große Testdateien in kleinere, wartbare Einheiten aufgeteilt
- **Typsicherheit:** Lokale Mock-Gruppen mit korrekten Typen
- **Wiederverwendbarkeit:** Mock-Factory kann von verschiedenen Tests genutzt werden

### 🔥 **Kritischer Erfolgsfaktor:**
Der **einmalige Import ohne FROM** in der Haupttestdatei ist **ZWINGEND ERFORDERLICH**, um die hoisted Mocks zu laden. Ohne diesen Import funktionieren die Mocks nicht!

---

## 📝 5. Alternative: __mocks__ vs. Expliziter Pfad

```typescript
// Option A: Automatische Erkennung (wenn im __mocks__ Verzeichnis)
// Vitest erkennt automatisch Mocks im __mocks__ Ordner

// Option B: Expliziter Pfad (wie in unserem Boilerplate)
import '@test/utils/mocks/ServiceName/ServiceName.ts'
```

**Empfehlung:** Expliziter Pfad für bessere Kontrolle und Vermeidung von Hoisting-Problemen! 🎯























<br><br>

---

<br><br>



# Option 2 

## 📋 Überblick der aktuellen Test-Architektur

Diese Struktur implementiert ein **zentralisiertes Mock-System** für die `MessageHandlerService` Tests mit modularen Sub-Dateien.

### 🏗️ Architektur-Prinzipien:
- **Zentrale Mock-Definitionen** in der Hauptdatei
- **Modulare Test-Suites** in separaten Dateien  
- **Konsistente Handler-Mocking** über alle Module hinweg
- **Einzelne MessageHandlerService-Instanzen** pro Test

---

## 📄 Hauptdatei: `MessageHandlerService.test.ts`

```typescript
// ==== DEPENDENCIES ====
import { describe, vi } from 'vitest'
import { ElectronMessage, ElectronMessageType } from '@main/models/ElectronMessage.ts'

// Import modularized test functions (alphabetically sorted)
import { runAppointmentsTests } from './MessageHandlerService-modules/appointments.ts'
import { runConfigurationsTests } from './MessageHandlerService-modules/configurations.ts'
import { runCustomerFindingsTests } from './MessageHandlerService-modules/customer-findings.ts'
import { runPatientsTests } from './MessageHandlerService-modules/patients.ts'
// ... weitere Imports

// ==== GLOBAL HANDLER MOCKS ====
// Diese Mocks werden von allen Modulen verwendet
vi.mock('@main/services/message-handlers/GetPatientsHandler.ts', () => ({
    handleGetPatients: vi.fn(async() => 
        Promise.resolve(new ElectronMessage(ElectronMessageType.getPatients))
    )
}))

vi.mock('@main/services/message-handlers/SyncPatientsHandler.ts', () => ({
    handleSyncPatients: vi.fn(async() => 
        Promise.resolve(new ElectronMessage(ElectronMessageType.syncPatients))
    )
}))

// ... weitere Handler-Mocks (insgesamt 16 Handler)

describe('[MessageHandlerService.ts] - src/main/services/MessageHandlerService.ts', () => {
    describe('handleMessages()', () => {
        // Execute all modularized test suites
        runPatientsTests()
        runConfigurationsTests()
        runCustomerFindingsTests()
        // ... weitere Test-Suites
    })
})
```

---

## 📄 Beispiel Sub-Modul: `patients.ts`

```typescript
// ==== DEPENDENCIES ====
import { beforeEach, describe, expect, it, vi } from 'vitest'
import { ElectronMessage, ElectronMessageType } from '@main/models/ElectronMessage.ts'
import * as getPatientDetailsHandler from '@main/services/message-handlers/GetPatientDetailsHandler.ts'
import * as getPatientsHandler from '@main/services/message-handlers/GetPatientsHandler.ts'
import * as syncPatientsHandler from '@main/services/message-handlers/SyncPatientsHandler.ts'
import { MessageHandlerService } from '@main/services/MessageHandlerService.ts'

// Handler mocks werden in der Hauptdatei definiert

export function runPatientsTests(): void {
    describe('Patients', () => {
        let messageHandlerService: MessageHandlerService

        beforeEach(() => {
            messageHandlerService = new MessageHandlerService()
        })

        describe('[✅ Success Cases]', () => {
            it('sollte Nachrichten korrekt an den handleGetPatients Handler weiterleiten', async() => {
                const inputMsg = new ElectronMessage(ElectronMessageType.getPatients)
                inputMsg.data = { some: 'data' }

                await messageHandlerService.handleMessages(inputMsg)

                expect(vi.mocked(getPatientsHandler.handleGetPatients)).toHaveBeenCalledTimes(1)
                expect(vi.mocked(getPatientsHandler.handleGetPatients))
                    .toHaveBeenCalledWith(inputMsg, expect.anything())
            })

            it('sollte Nachrichten korrekt an den handleSyncPatients Handler weiterleiten', async() => {
                const inputMsg = new ElectronMessage(ElectronMessageType.syncPatients)
                await messageHandlerService.handleMessages(inputMsg)
                
                expect(vi.mocked(syncPatientsHandler.handleSyncPatients)).toHaveBeenCalledTimes(1)
                expect(vi.mocked(syncPatientsHandler.handleSyncPatients))
                    .toHaveBeenCalledWith(inputMsg, expect.anything())
            })
        })

        describe('[❌ Error Cases]', () => {
            it('sollte den Fehler als string verarbeiten, wenn ein Handler einen Nicht-Error-Wert wirft', async() => {
                const inputMsg = new ElectronMessage(ElectronMessageType.getPatients)
                const mockErrorValue = 'Mein String Fehler'
                
                vi.mocked(getPatientsHandler.handleGetPatients).mockRejectedValueOnce(mockErrorValue)

                const outputMsg = await messageHandlerService.handleMessages(inputMsg)

                expect(outputMsg.success).toBe(false)
                expect(outputMsg.errorMsg).toBe(String(mockErrorValue))
            })
        })
    })
}
```

---

## 🔧 Wichtige Patterns & Konventionen

### **Mock-Verhalten:**
- **Zentrale Definition:** Alle `vi.mock()` Calls in der Hauptdatei **vor** den Modul-Imports
- **Import Pattern:** `import * as handlerName` für alle Handler in Sub-Modulen
- **Assertions:** `vi.mocked(handlerName.functionName)` für Mock-Verifikationen

### **Test-Setup:**
- **beforeEach:** Erstellt **neue Instanz** von `MessageHandlerService` (keine Mock-Resets)
- **Isolation:** Jeder Test erhält eine frische Service-Instanz
- **Struktur:** `[✅ Success Cases]` und `[❌ Error Cases]` describe-Blöcke

### **Mock-Assertions Beispiele:**
```typescript
// Aufruf-Verifikation
expect(vi.mocked(handlerName.functionName)).toHaveBeenCalledTimes(1)

// Parameter-Verifikation  
expect(vi.mocked(handlerName.functionName))
    .toHaveBeenCalledWith(inputMsg, expect.anything())

// Mock-Behavior für Error-Tests
vi.mocked(handlerName.functionName).mockRejectedValueOnce(new Error('Test Error'))
```

### **Datei-Naming:**
- Hauptdatei: `MessageHandlerService.test.ts`
- Sub-Module: `MessageHandlerService-modules/{category}.ts`
- Export-Funktion: `run{Category}Tests()`

---

## ✅ Vorteile dieser Struktur

- **🎯 Zentrale Mock-Kontrolle:** Alle Mocks an einem Ort verwaltet
- **📦 Modulare Tests:** Bessere Organisation und Wartbarkeit
- **🔄 Konsistenz:** Einheitliche Patterns über alle Test-Module
- **⚡ Performance:** Keine redundanten Mock-Definitionen
- **🛠️ Skalierbarkeit:** Einfaches Hinzufügen neuer Handler/Tests




















<br><br>

---

<br><br>


# Option 2 - injectHoistedMocks()

## 🎯 Überblick

Dieses Refactoring implementiert eine **zentrale Mock-Factory** für MessageHandler-Tests mit **vi.hoisted()** Pattern und **injectHoistedMocks()** Methode zur Vermeidung von Hoisting-Problemen.

## 🏗️ Architektur

```
📁 test/utils/mocks/
  └── MessageHandlerMockFactory.ts     # Zentrale Mock-Factory
📁 test/unit/main/services/
  ├── MessageHandlerService.test.ts    # Haupttest mit vi.hoisted() 
  └── MessageHandlerService-modules/
      └── appointments.ts         # Modularisierte Tests
```

---

## 📄 Boilerplate Code

### 1. Mock-Factory (`test/utils/mocks/MessageHandlerMockFactory.ts`)

```typescript
// ==== DEPENDENCIES ====
import { type MockedFunction } from 'vitest'
import { ElectronMessage } from '@main/models/ElectronMessage.ts'
import { MessageHandlerService } from '@main/services/MessageHandlerService.ts'

// ==== TYPES ====
type MessageHandlerFunction = (
    message: Readonly<ElectronMessage>, 
    webContents: Readonly<Electron.WebContents>
) => Promise<ElectronMessage>

interface IMessageHandlerMocks {
    // Appointments
    handleGetAppointmentsForToday: MockedFunction<MessageHandlerFunction>
    handleGetAppointments: MockedFunction<MessageHandlerFunction>
    // ... weitere Handler
}

interface IMessageHandlerTestSuite {
    service: MessageHandlerService
    mocks: IMessageHandlerMocks
    resetMocks: () => void
}

// ==== FACTORY CLASS ====
export class MessageHandlerMockFactory {
    private static _instance: MessageHandlerMockFactory
    private _hoistedMocks: IMessageHandlerMocks | null = null

    private constructor() {
        // Private constructor für Singleton Pattern
    }

    public static getInstance(): MessageHandlerMockFactory {
        if (!MessageHandlerMockFactory._instance) {
            MessageHandlerMockFactory._instance = new MessageHandlerMockFactory()
        }
        return MessageHandlerMockFactory._instance
    }

    /**
     * Injiziert die vi.hoisted Mocks in die Factory
     * Diese Methode MUSS von der Haupttest-Datei nach dem vi.hoisted() Aufruf verwendet werden
     */
    public injectHoistedMocks(mocks: Readonly<IMessageHandlerMocks>): void {
        this._hoistedMocks = mocks as IMessageHandlerMocks
    }

    /**
     * Erstellt eine neue Test-Suite mit Service-Instanz und Mock-Zugriff
     */
    public createTestSuite(): IMessageHandlerTestSuite {
        if (this._hoistedMocks === null) {
            throw new Error('Hoisted mocks müssen zuerst mit injectHoistedMocks() injiziert werden')
        }

        const service = new MessageHandlerService()
        const mocks = this._hoistedMocks

        return {
            service,
            mocks,
            resetMocks: (): void => {
                Object.values(mocks).forEach(mock => {
                    if (mock !== null && typeof mock.mockClear === 'function') {
                        mock.mockClear()
                    }
                })
            }
        }
    }

    // Weitere Methoden: configureMock(), resetMock(), getMockStats()...
}

export type { IMessageHandlerMocks, IMessageHandlerTestSuite }
```

### 2. Haupttest-Datei (`test/unit/main/services/MessageHandlerService.test.ts`)

```typescript
// ==== DEPENDENCIES ====
import { describe, vi } from 'vitest'
import { ElectronMessage, ElectronMessageType } from '@main/models/ElectronMessage.ts'
import { MessageHandlerMockFactory } from '../../../utils/mocks/MessageHandlerMockFactory.ts'

// Import modularized test functions
import { runAppointmentsTests } from './MessageHandlerService-modules/appointments.ts'
// ... weitere Test-Importe

// ==== HOISTED MOCKS ====
// Diese Mocks MÜSSEN vor den vi.mock() Aufrufen mit vi.hoisted definiert werden
const mockHandlers = vi.hoisted(() => ({
    // Appointments
    handleGetAppointmentsForToday: vi.fn(async() => 
        Promise.resolve(new ElectronMessage(ElectronMessageType.getAppointmentsForToday))
    ),
    handleGetAppointments: vi.fn(async() => 
        Promise.resolve(new ElectronMessage(ElectronMessageType.getAppointments))
    ),
    
    // Configurations
    handleCheckConfiguration: vi.fn(async() => 
        Promise.resolve(new ElectronMessage(ElectronMessageType.checkConfiguration))
    ),
    // ... weitere Mock-Handler
}))

// ==== MOCK FACTORY SETUP ====
// Initialisiere die Mock-Factory und injiziere die gehoisteten Mocks
const mockFactory = MessageHandlerMockFactory.getInstance()
mockFactory.injectHoistedMocks(mockHandlers)

// ==== HANDLER MOCKS ====
// Diese vi.mock Aufrufe verwenden nun die korrekt gehoisteten Mocks
vi.mock('@main/services/message-handlers/GetAppointmentsForTodayHandler.ts', () => ({
    handleGetAppointmentsForToday: mockHandlers.handleGetAppointmentsForToday
}))

vi.mock('@main/services/message-handlers/GetAppointmentsHandler.ts', () => ({
    handleGetAppointments: mockHandlers.handleGetAppointments
}))

// ... weitere vi.mock() Aufrufe

// ==== TESTS ====
describe('[MessageHandlerService.ts] - src/main/services/MessageHandlerService.ts', () => {
    describe('handleMessages()', () => {
        // Execute all modularized test suites
        // Die Mock-Factory ist global verfügbar für alle Test-Suites
        runAppointmentsTests()
        // ... weitere Test-Suite Aufrufe
    })
})

// ==== GLOBAL EXPORT ====
// Exportiere die Mock-Factory für die Verwendung in Test-Modulen
export { mockFactory }
```

### 3. Modularisierte Tests (`test/unit/main/services/MessageHandlerService-modules/appointments.ts`)

```typescript
// ==== DEPENDENCIES ====
import { beforeEach, describe, expect, it } from 'vitest'
import { ElectronMessage, ElectronMessageType } from '@main/models/ElectronMessage.ts'
import { 
    MessageHandlerMockFactory,
    type IMessageHandlerTestSuite 
} from '../../../../utils/mocks/MessageHandlerMockFactory.ts'

export function runAppointmentsTests(): void {
    describe('Appointments', () => {
        let testSuite: IMessageHandlerTestSuite

        beforeEach(() => {
            // Erstelle eine frische Test-Suite mit Service und Mocks
            const mockFactory = MessageHandlerMockFactory.getInstance()
            testSuite = mockFactory.createTestSuite()
            
            // Setze alle Mocks zurück für eine saubere Testumgebung
            testSuite.resetMocks()
        })

        describe('[✅ Success Cases]', () => {
            it('sollte Nachrichten korrekt an den handleGetAppointmentsForToday Handler weiterleiten', async() => {
                const inputMsg = new ElectronMessage(ElectronMessageType.getAppointmentsForToday)
                
                await testSuite.service.handleMessages(inputMsg)
                
                // Verwende die Mock-Factory für Assertions - keine direkten vi.mocked Aufrufe mehr
                expect(testSuite.mocks.handleGetAppointmentsForToday).toHaveBeenCalledTimes(1)
                expect(testSuite.mocks.handleGetAppointmentsForToday)
                    .toHaveBeenCalledWith(inputMsg, expect.anything())
            })
        })

        describe('[🔧 Advanced Mock Usage Examples]', () => {
            it('sollte spezifisches Mock-Verhalten für Error-Handling demonstrieren', async() => {
                const inputMsg = new ElectronMessage(ElectronMessageType.getAppointmentsForToday)
                
                // Konfiguriere spezifisches Mock-Verhalten mit der Factory
                const mockFactory = MessageHandlerMockFactory.getInstance()
                mockFactory.configureMock('handleGetAppointmentsForToday', async() => {
                    const errorMsg = new ElectronMessage(ElectronMessageType.getAppointmentsForToday)
                    errorMsg.success = false
                    errorMsg.errorMsg = 'Database connection failed'
                    return Promise.resolve(errorMsg)
                })

                const result = await testSuite.service.handleMessages(inputMsg)
                
                expect(result.success).toBe(false)
                expect(result.errorMsg).toBe('Database connection failed')
                expect(testSuite.mocks.handleGetAppointmentsForToday).toHaveBeenCalledTimes(1)
            })
        })
    })
}
```

---

## 🔄 Workflow

1. **vi.hoisted()** definiert alle Mocks in der Haupttest-Datei
2. **injectHoistedMocks()** übergibt die Mocks an die Factory
3. **vi.mock()** Aufrufe verwenden die gehoisteten Mock-Referenzen
4. **Modularisierte Tests** verwenden die Factory für Service-Instanzen und Mock-Zugriff
5. **testSuite.mocks** stellt typisierte Mock-Referenzen bereit

## ✅ Vorteile

- ✅ **Keine Hoisting-Probleme** mehr
- ✅ **Zentrale Mock-Verwaltung** über Factory
- ✅ **Typsicherheit** bei Mock-Zugriff  
- ✅ **Saubere Test-Isolation** durch beforeEach()
- ✅ **Modulare Test-Struktur** beibehalten
- ✅ **Erweiterte Mock-Konfiguration** möglich

