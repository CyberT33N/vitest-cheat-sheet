# vi.mock() - Import des Mocks mit Callback

- Dieses Beispiel hier benutzt zwar den gleichen Ordnernamen **__mox__** für den **MOC-Ordner**. Er ist aber nicht äquivalent zu dem auf der **Routebene**, welcher automatisch gemockt wird, wenn man die **MOC** mit dem jeweiligen **Modul** aufruft. Wir haben es nur sinnbildlich gleich genannt.







## Beispiel

### Neue Dateistruktur

```
test/
├── __mocks__/
│   └── @pinecone-database/
│       └── pinecone.ts          # ✅ Zentrale Mock-Datei
└── unit/
    └── src/
        └── services/
            └── pinecone-service.test.ts  # ✅ Saubere Test-Datei
```

### 1. Mock-Factory Klasse (`test/__mocks__/@pinecone-database/pinecone.ts`)

```typescript
/*
███████████████████████████████████████████████████████████████████████████████
██******************** PRESENTED BY t33n Software ***************************██
██                                                                           ██
██                  ████████╗██████╗ ██████╗ ███╗   ██╗                      ██
██                  ╚══██╔══╝╚════██╗╚════██╗████╗  ██║                      ██
██                     ██║    █████╔╝ █████╔╝██╔██╗ ██║                      ██
██                     ██║    ╚═══██╗ ╚═══██╗██║╚██╗██║                      ██
██                     ██║   ██████╔╝██████╔╝██║ ╚████║                      ██
██                     ╚═╝   ╚═════╝ ╚═════╝ ╚═╝  ╚═══╝                      ██
██                                                                           ██
███████████████████████████████████████████████████████████████████████████████
███████████████████████████████████████████████████████████████████████████████
*/

// ==== Imports ====
import {
    Index,
    type PineconeConfiguration,
    Pinecone,
    Inference
} from '@pinecone-database/pinecone'
import { vi, type MockedFunction } from 'vitest'

// ==== Types ====
type PineconeModule = typeof import('@pinecone-database/pinecone')

// ==== Mock Factory ====
export class PineconeMockFactory {
    // Individual mock functions to avoid unbound method issues
    public mockNamespaceFn: MockedFunction<(...args: readonly unknown[]) => unknown>
    public mockDescribeIndexStatsFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    public mockUpsertFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    public mockQueryFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    public mockDeleteManyFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    public mockDeleteAllFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    public mockDeleteNamespaceFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    public mockListNamespacesFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    
    // Pinecone instance mock functions
    public mockDescribeIndexFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    public mockCreateIndexFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    public mockListIndexesFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    public mockIndexFn: MockedFunction<(...args: readonly unknown[]) => unknown>
    
    // Mock instances
    public mockPineconeInstance: Pinecone
    public mockIndexInstance: Index
    public mockNamespaceInstance: Index
    
    // Mock module reference
    public mockPinecone: PineconeModule | undefined
    
    public constructor() {
        // Initialize all mock functions FIRST
        this.mockUpsertFn = vi.fn()
        this.mockQueryFn = vi.fn()
        this.mockDeleteManyFn = vi.fn()
        this.mockDeleteAllFn = vi.fn()
        this.mockDeleteNamespaceFn = vi.fn()
        this.mockListNamespacesFn = vi.fn()
        this.mockNamespaceFn = vi.fn()
        this.mockDescribeIndexStatsFn = vi.fn()
        this.mockDescribeIndexFn = vi.fn()
        this.mockCreateIndexFn = vi.fn()
        this.mockListIndexesFn = vi.fn()
        this.mockIndexFn = vi.fn()
        
        // Create mock instances AFTER mock functions are initialized
        this.mockIndexInstance = this._createMockIndexObject()
        this.mockNamespaceInstance = this.mockIndexInstance // Same object for namespace
        this.mockPineconeInstance = this._createMockPineconeClient()
        
        // Configure mock functions to return appropriate instances
        this.mockNamespaceFn.mockReturnValue(this.mockNamespaceInstance)
        this.mockIndexFn.mockReturnValue(this.mockIndexInstance)
    }
    
    public resetAllMocks(): void {
        // Reset all mock functions
        this.mockUpsertFn.mockReset()
        this.mockQueryFn.mockReset()
        this.mockDeleteManyFn.mockReset()
        this.mockDeleteAllFn.mockReset()
        this.mockDeleteNamespaceFn.mockReset()
        this.mockListNamespacesFn.mockReset()
        this.mockNamespaceFn.mockReset()
        this.mockDescribeIndexStatsFn.mockReset()
        this.mockDescribeIndexFn.mockReset()
        this.mockCreateIndexFn.mockReset()
        this.mockListIndexesFn.mockReset()
        this.mockIndexFn.mockReset()
        
        // Reconfigure return values
        this.mockNamespaceFn.mockReturnValue(this.mockNamespaceInstance)
        this.mockIndexFn.mockReturnValue(this.mockIndexInstance)
    }
    
    private _createMockIndexObject(): Index {
        return {
            // Core vector operations - use the pre-created mock functions
            namespace: this.mockNamespaceFn,
            upsert: this.mockUpsertFn,
            fetch: vi.fn(),
            query: this.mockQueryFn,
            update: vi.fn(),
            
            // Delete operations - use the pre-created mock functions
            deleteAll: this.mockDeleteAllFn,
            deleteMany: this.mockDeleteManyFn,
            deleteOne: vi.fn(),
            
            // Stats and info - use the pre-created mock function
            describeIndexStats: this.mockDescribeIndexStatsFn,
            listPaginated: vi.fn(),
            
            // Integrated records (new features)
            upsertRecords: vi.fn(),
            searchRecords: vi.fn(),
            
            // Import operations
            startImport: vi.fn(),
            listImports: vi.fn(),
            describeImport: vi.fn(),
            cancelImport: vi.fn(),
            
            // Namespace operations - use the pre-created mock functions
            listNamespaces: this.mockListNamespacesFn,
            describeNamespace: vi.fn(),
            deleteNamespace: this.mockDeleteNamespaceFn,
            
            // Hidden/Private methods from Index class
            _deleteMany: vi.fn(),
            _deleteOne: vi.fn(),
            _describeIndexStats: vi.fn(),
            _listPaginated: vi.fn(),
            _deleteAll: vi.fn(),
            _fetchCommand: vi.fn(),
            _queryCommand: vi.fn(),
            _updateCommand: vi.fn(),
            _upsertCommand: vi.fn(),
            _upsertRecordsCommand: vi.fn(),
            _searchRecordsCommand: vi.fn(),
            _startImportCommand: vi.fn(),
            _listImportsCommand: vi.fn(),
            _describeImportCommand: vi.fn(),
            _cancelImportCommand: vi.fn(),
            _listNamespacesCommand: vi.fn(),
            _describeNamespaceCommand: vi.fn(),
            _deleteNamespaceCommand: vi.fn(),
            
            // Internal properties
            config: {},
            target: 'mock-target'
        } as unknown as Index
    }
    
    private _createMockPineconeClient(): Pinecone {
        return {
            // Private methods (hidden)
            _configureIndex: vi.fn(),
            _createCollection: vi.fn(),
            _createIndex: vi.fn(),
            _createIndexForModel: vi.fn(),
            _describeCollection: vi.fn(),
            _describeIndex: vi.fn(),
            _deleteCollection: vi.fn(),
            _deleteIndex: vi.fn(),
            _listCollections: vi.fn(),
            _listIndexes: vi.fn(),
            _createAssistant: vi.fn(),
            _deleteAssistant: vi.fn(),
            _updateAssistant: vi.fn(),
            _describeAssistant: vi.fn(),
            _listAssistants: vi.fn(),
            _createBackup: vi.fn(),
            _createIndexFromBackup: vi.fn(),
            _describeBackup: vi.fn(),
            _describeRestoreJob: vi.fn(),
            _deleteBackup: vi.fn(),
            _listBackups: vi.fn(),
            _listRestoreJobs: vi.fn(),
            
            // Public inference property
            inference: {} as Inference,
            
            // Internal methods
            _readEnvironmentConfig: vi.fn(),
            config: {} as PineconeConfiguration,
            _checkForBrowser: vi.fn(),
            getConfig: vi.fn(),
            
            // Public index management methods - use pre-created mock functions
            describeIndex: this.mockDescribeIndexFn,
            listIndexes: this.mockListIndexesFn,
            createIndex: this.mockCreateIndexFn,
            createIndexForModel: vi.fn(),
            deleteIndex: vi.fn(),
            configureIndex: vi.fn(),
            
            // Public collection management methods
            createCollection: vi.fn(),
            listCollections: vi.fn(),
            deleteCollection: vi.fn(),
            describeCollection: vi.fn(),
            
            // Public assistant management methods
            createAssistant: vi.fn(),
            deleteAssistant: vi.fn(),
            describeAssistant: vi.fn(),
            listAssistants: vi.fn(),
            updateAssistant: vi.fn(),
            
            // Public backup management methods
            createBackup: vi.fn(),
            createIndexFromBackup: vi.fn(),
            describeBackup: vi.fn(),
            describeRestoreJob: vi.fn(),
            deleteBackup: vi.fn(),
            listBackups: vi.fn(),
            listRestoreJobs: vi.fn(),
            
            // Public index targeting methods - use pre-created mock function
            index: this.mockIndexFn,
            Index: vi.fn(),
            
            // Public assistant targeting methods
            assistant: vi.fn(),
            Assistant: vi.fn()
        } as unknown as Pinecone
    }
}

// ==== Global Mock Factory Instance ====
export const pineconeMockFactory = new PineconeMockFactory()

// ==== Mock Module Export ====
export const mockPineconeModule = async(): Promise<PineconeModule> => {
    const original = await vi.importActual<PineconeModule>('@pinecone-database/pinecone')
    
    const mockedModule = {
        ...original,
        // Mock the Pinecone constructor to return our mock instance
        Pinecone: vi.fn().mockImplementation(() => pineconeMockFactory.mockPineconeInstance),
        // Export other classes as-is but could be mocked if needed
        Index: original.Index,
        Inference: original.Inference
    }
    
    // Store reference to the mocked module in the factory for test access
    pineconeMockFactory.mockPinecone = mockedModule
    
    return mockedModule
}

// ==== Default Export ====
export default mockPineconeModule 
```

### 2. Vereinfachte Test-Datei

```typescript
// ✅ KORREKT - Saubere Test-Struktur
import { pineconeMockFactory } from '@test/__mocks__/@pinecone-database/pinecone.js'

// Mock-Setup mit Inline-Import (löst Hoisting-Problem)
vi.mock('@pinecone-database/pinecone', async () => {
    const { mockPineconeModule } = await import('@test/__mocks__/@pinecone-database/pinecone.js')
    return mockPineconeModule()
})

describe('PineconeService() - Unit Tests', () => {
    let service: PineconeService
    let mockIndexInstance: Index
    
    // Direkte Referenzen auf Mock-Funktionen (keine unbound methods)
    let mockNamespaceFn: MockedFunction<(...args: readonly unknown[]) => unknown>
    let mockUpsertFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    // ...

    beforeEach(() => {
        // Reset aller Mocks
        pineconeMockFactory.resetAllMocks()
        
        // Direkte Zuweisung von der Factory (keine Getter-Funktionen nötig)
        mockIndexInstance = pineconeMockFactory.mockIndexInstance
        mockNamespaceFn = pineconeMockFactory.mockNamespaceFn
        mockUpsertFn = pineconeMockFactory.mockUpsertFn
        // ...
        
        service = createStandardPineconeService()
    })

    describe('Constructor', () => {
        it('sollte korrekt initialisieren', () => {
            // Sichere Null-Checks anstelle von Non-null-Assertions
            expect(pineconeMockFactory.mockPinecone).toBeDefined()
            expect(pineconeMockFactory.mockPinecone?.Pinecone).toHaveBeenCalledWith({ apiKey })
        })
    })
})
```

## Wichtige Lösungsansätze

### 1. Hoisting-Problem lösen

**Problem**: `vi.mock()` benötigt zur Compile-Zeit verfügbare Funktionen.

**Lösung**: Inline-Import in der Mock-Definition mit Callback:
```typescript
// ✅ KORREKT
vi.mock('@pinecone-database/pinecone', async () => {
    const { mockPineconeModule } = await import('@test/__mocks__/@pinecone-database/pinecone.js')
    return mockPineconeModule()
})

// ❌ FALSCH - Verursacht Hoisting-Fehler
import { mockPineconeModule } from '@test/__mocks__/@pinecone-database/pinecone.js'
vi.mock('@pinecone-database/pinecone', mockPineconeModule)
```

### 2. Unbound Method Errors vermeiden

**Problem**: ESLint-Regel `@typescript-eslint/unbound-method` verhindert direkte Methodenreferenzen.

**Lösung**: Mock-Funktionen als Properties der Factory-Klasse:
```typescript
// ✅ KORREKT - Direkte Property-Zugriffe
export class PineconeMockFactory {
    public mockUpsertFn: MockedFunction<...>
    
    private _createMockIndexObject(): Index {
        return {
            upsert: this.mockUpsertFn,  // ✅ Bound method
        }
    }
}

// Test-Verwendung
mockUpsertFn = pineconeMockFactory.mockUpsertFn  // ✅ Keine unbound method
```

### 3. Type-Safety gewährleisten

**Problem**: `undefined`-Zugriffe auf Mock-Properties.

**Lösung**: Sichere Null-Checks:
```typescript
// ✅ KORREKT - Sichere Null-Checks
expect(pineconeMockFactory.mockPinecone).toBeDefined()
expect(pineconeMockFactory.mockPinecone?.Pinecone).toHaveBeenCalledWith(config)

// ❌ FALSCH - Non-null-Assertion (ESLint-Fehler)
expect(pineconeMockFactory.mockPinecone!.Pinecone).toHaveBeenCalledWith(config)
```

## Vorteile der neuen Struktur

### ✅ Vorteile

1. **Modularität**: Mock-Code ist in separater Datei organisiert
2. **Wiederverwendbarkeit**: Factory kann in mehreren Test-Dateien verwendet werden
3. **Wartbarkeit**: Klare Trennung von Mock-Logic und Test-Logic
4. **Type-Safety**: Vollständige TypeScript-Unterstützung
5. **ESLint-Konformität**: Keine unbound-method Warnungen
6. **Einfache Erweiterung**: Neue Mock-Funktionen einfach hinzufügbar


## Anwendung für neue Tests

### Template für neue Mock-basierte Tests

```typescript
// 1. Import der Mock-Factory
import { pineconeMockFactory } from '@test/__mocks__/@pinecone-database/pinecone.js'

// 2. Mock-Setup mit Inline-Import
vi.mock('@pinecone-database/pinecone', async () => {
    const { mockPineconeModule } = await import('@test/__mocks__/@pinecone-database/pinecone.js')
    return mockPineconeModule()
})

// 3. Test-Setup
describe('YourService Tests', () => {
    let service: YourService
    
    // 4. Mock-Referenzen definieren
    let mockUpsertFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    
    beforeEach(() => {
        // 5. Mocks zurücksetzen
        pineconeMockFactory.resetAllMocks()
        
        // 6. Mock-Referenzen zuweisen
        mockUpsertFn = pineconeMockFactory.mockUpsertFn
        
        // 7. Service erstellen
        service = new YourService()
    })
    
    it('should work correctly', () => {
        // 8. Mock-Verhalten konfigurieren
        mockUpsertFn.mockResolvedValue(undefined)
        
        // 9. Test ausführen und verifizieren
        // ...
        
        expect(mockUpsertFn).toHaveBeenCalledWith(expectedArgs)
    })
})
```


<details>
























<br><br>
<br><br>


# Troubshooting



### ❌ Problem: `importMock()` Inside `vi.mock()` = 💥 Infinite Recursion

Calling `importMock()` **inside** a `vi.mock()` block is a trap:
It tries to load the very module you're currently mocking → triggers `vi.mock()` again → **infinite loop** → boom.

---

### ✅ Solution #1: Use `importOriginal()`

Never use `importMock()` inside `vi.mock()`.
Instead, Vitest provides `importOriginal or importActual` exactly for this purpose: