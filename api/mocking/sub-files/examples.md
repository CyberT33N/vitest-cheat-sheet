# Option 1 

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
import { runAppointmentsTests } from './MessageHandlerService-modules/appointments.test.ts'
import { runConfigurationsTests } from './MessageHandlerService-modules/configurations.test.ts'
import { runCustomerFindingsTests } from './MessageHandlerService-modules/customer-findings.test.ts'
import { runPatientsTests } from './MessageHandlerService-modules/patients.test.ts'
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

## 📄 Beispiel Sub-Modul: `patients.test.ts`

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
- Sub-Module: `MessageHandlerService-modules/{category}.test.ts`
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
      └── appointments.test.ts         # Modularisierte Tests
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
import { runAppointmentsTests } from './MessageHandlerService-modules/appointments.test.ts'
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

### 3. Modularisierte Tests (`test/unit/main/services/MessageHandlerService-modules/appointments.test.ts`)

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

