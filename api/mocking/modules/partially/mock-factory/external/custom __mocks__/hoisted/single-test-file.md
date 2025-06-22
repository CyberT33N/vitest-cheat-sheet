## Vereinfachtes Beispiel: Zentrale Mock-Datei für einen einzigen Service

Dieses Beispiel reduziert die Architektur auf das absolute Minimum. Es zeigt, wie man eine **einzige Methode** eines einfachen Services mit einer zentralen, gehoisteten Mock-Datei mockt. Der Fokus liegt auf der **Klarheit der Architektur** und dem korrekten Import-Verhalten.

### 🎯 Überblick der Architektur

Wir verwenden eine externe Mock-Datei mit `vi.hoisted()`, um einen Mock vorzubereiten. Dieser Mock wird dann in der Testdatei durch einen speziellen Import aktiviert, um eine einzelne Service-Methode zu ersetzen.

---

### 🗂️ Dateistruktur

```
test/
├──__mocks__/SimpleCalculator/
│   └── SimpleCalculator.ts       # 🔑 ZENTRALE MOCK-DATEI
└── unit/main/services/
    └── SimpleCalculator.test.ts  # 🏠 TESTDATEI
```
*(Annahme: Der echte Service liegt unter `src/main/services/SimpleCalculator.ts`)*

---

### 🔑 1. Zentrale Mock-Datei (`test/__mocks__/SimpleCalculator/SimpleCalculator.ts`)

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
// eslint-disable-next-line import/no-duplicates, no-duplicate-imports
import '@test/__mocks__/SimpleCalculator/SimpleCalculator.ts'

// Regulärer Import, um auf die Mock-Instanz zuzugreifen und Assertions zu machen.
// eslint-disable-next-line import/no-duplicates, no-duplicate-imports
import { calculatorMocks } from '@test/__mocks__/SimpleCalculator/SimpleCalculator.ts'

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
import '@test/__mocks__/SimpleCalculator/SimpleCalculator.ts'

// ❌ FALSCH: Die Mocks werden NICHT aktiviert
import { calculatorMocks } from '@test/__mocks__/SimpleCalculator/SimpleCalculator.ts'
```

--- 






<br><br>

---

<br><br>


# Example Pincecone

```typescript
// ==== DEPENDENCIES ====
import {
    Index,
    type PineconeConfiguration,
    Pinecone,
    Inference
} from '@pinecone-database/pinecone'
import { vi, type Mock } from 'vitest'

// ==== TYPES ====
type PineconeModule = typeof import('@pinecone-database/pinecone')

// ==== HOISTED MOCK SETUP ====
const mockProvider = vi.hoisted(() => {
    // Index-level mock functions
    const mockNamespaceFn = vi.fn()
    const mockDescribeIndexStatsFn = vi.fn()
    const mockUpsertFn = vi.fn()
    const mockQueryFn = vi.fn()
    const mockDeleteManyFn = vi.fn()
    const mockDeleteAllFn = vi.fn()
    const mockDeleteNamespaceFn = vi.fn()
    const mockListNamespacesFn = vi.fn()
    
    // Pinecone instance mock functions
    const mockDescribeIndexFn = vi.fn()
    const mockCreateIndexFn = vi.fn()
    const mockListIndexesFn = vi.fn()
    const mockIndexFn = vi.fn()
    
    // Create mock Index instance
    const createMockIndexObject = (): Index => {
        return {
            // Core vector operations
            namespace: mockNamespaceFn,
            upsert: mockUpsertFn,
            fetch: vi.fn(),
            query: mockQueryFn,
            update: vi.fn(),
            
            // Delete operations
            deleteAll: mockDeleteAllFn,
            deleteMany: mockDeleteManyFn,
            deleteOne: vi.fn(),
            
            // Stats and info
            describeIndexStats: mockDescribeIndexStatsFn,
            listPaginated: vi.fn(),
            
            // Integrated records (new features)
            upsertRecords: vi.fn(),
            searchRecords: vi.fn(),
            
            // Import operations
            startImport: vi.fn(),
            listImports: vi.fn(),
            describeImport: vi.fn(),
            cancelImport: vi.fn(),
            
            // Namespace operations
            listNamespaces: mockListNamespacesFn,
            describeNamespace: vi.fn(),
            deleteNamespace: mockDeleteNamespaceFn,
            
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
    
    // Create mock Pinecone client
    const createMockPineconeClient = (): Pinecone => {
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
            
            // Public index management methods
            describeIndex: mockDescribeIndexFn,
            listIndexes: mockListIndexesFn,
            createIndex: mockCreateIndexFn,
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
            
            // Public index targeting methods
            index: mockIndexFn,
            Index: vi.fn(),
            
            // Public assistant targeting methods
            assistant: vi.fn(),
            Assistant: vi.fn()
        } as unknown as Pinecone
    }
    
    // Create mock instances
    const mockIndexInstance = createMockIndexObject()
    const mockNamespaceInstance = mockIndexInstance // Same object for namespace
    const mockPineconeInstance = createMockPineconeClient()
    
    // Configure mock functions to return appropriate instances
    mockNamespaceFn.mockReturnValue(mockNamespaceInstance)
    mockIndexFn.mockReturnValue(mockIndexInstance)
    
    return {
        // Mock functions
        mockNamespaceFn,
        mockDescribeIndexStatsFn,
        mockUpsertFn,
        mockQueryFn,
        mockDeleteManyFn,
        mockDeleteAllFn,
        mockDeleteNamespaceFn,
        mockListNamespacesFn,
        mockDescribeIndexFn,
        mockCreateIndexFn,
        mockListIndexesFn,
        mockIndexFn,
        
        // Mock instances
        mockPineconeInstance,
        mockIndexInstance,
        mockNamespaceInstance,
        
        // Utility functions
        createMockIndexObject,
        createMockPineconeClient
    }
})

// Get the original module to create the mocked version
const original = await vi.importActual<PineconeModule>('@pinecone-database/pinecone')

// Create the mocked module object
const mockedPineconeModule = {
    ...original,
    // Mock the Pinecone constructor to return our mock instance
    Pinecone: vi.fn().mockImplementation(() => mockProvider.mockPineconeInstance),
    // Export other classes as-is but could be mocked if needed
    Index: original.Index,
    Inference: original.Inference
}

// ==== MODULE MOCKS (HOISTED) ====
vi.mock('@pinecone-database/pinecone', () => mockedPineconeModule)

// ==== MOCK FACTORY ====
export class PineconeMockFactory {
    private static _instance: PineconeMockFactory | undefined
    
    // Direct mock properties
    public readonly mockNamespaceFn = mockProvider.mockNamespaceFn
    public readonly mockDescribeIndexStatsFn = mockProvider.mockDescribeIndexStatsFn  
    public readonly mockUpsertFn = mockProvider.mockUpsertFn
    public readonly mockQueryFn = mockProvider.mockQueryFn
    public readonly mockDeleteManyFn = mockProvider.mockDeleteManyFn
    public readonly mockDeleteAllFn = mockProvider.mockDeleteAllFn
    public readonly mockDeleteNamespaceFn = mockProvider.mockDeleteNamespaceFn
    public readonly mockListNamespacesFn = mockProvider.mockListNamespacesFn
    public readonly mockDescribeIndexFn = mockProvider.mockDescribeIndexFn
    public readonly mockCreateIndexFn = mockProvider.mockCreateIndexFn
    public readonly mockListIndexesFn = mockProvider.mockListIndexesFn
    public readonly mockIndexFn = mockProvider.mockIndexFn
    
    // Mock instances
    public readonly mockPineconeInstance = mockProvider.mockPineconeInstance
    public readonly mockIndexInstance = mockProvider.mockIndexInstance
    public readonly mockNamespaceInstance = mockProvider.mockNamespaceInstance
    
    // Mock module reference - FIXING THE BREAKING CHANGE
    public readonly mockPinecone = mockedPineconeModule
    
    public static getInstance(): PineconeMockFactory {
        PineconeMockFactory._instance ??= new PineconeMockFactory()
        return PineconeMockFactory._instance
    }
    
    public resetAllMocks(): void {
        // Reset all mock functions
        mockProvider.mockUpsertFn.mockReset()
        mockProvider.mockQueryFn.mockReset()
        mockProvider.mockDeleteManyFn.mockReset()
        mockProvider.mockDeleteAllFn.mockReset()
        mockProvider.mockDeleteNamespaceFn.mockReset()
        mockProvider.mockListNamespacesFn.mockReset()
        mockProvider.mockNamespaceFn.mockReset()
        mockProvider.mockDescribeIndexStatsFn.mockReset()
        mockProvider.mockDescribeIndexFn.mockReset()
        mockProvider.mockCreateIndexFn.mockReset()
        mockProvider.mockListIndexesFn.mockReset()
        mockProvider.mockIndexFn.mockReset()
        
        // Reconfigure return values
        mockProvider.mockNamespaceFn.mockReturnValue(mockProvider.mockNamespaceInstance)
        mockProvider.mockIndexFn.mockReturnValue(mockProvider.mockIndexInstance)
    }
    
    // Convenience methods for getting specific mock groups
    public getIndexMocks(): {
        mockNamespaceFn: Mock,
        mockUpsertFn: Mock,
        mockQueryFn: Mock,
        mockDeleteManyFn: Mock,
        mockDeleteAllFn: Mock,
        mockDescribeIndexStatsFn: Mock
        } {
        return {
            mockNamespaceFn: mockProvider.mockNamespaceFn,
            mockUpsertFn: mockProvider.mockUpsertFn,
            mockQueryFn: mockProvider.mockQueryFn,
            mockDeleteManyFn: mockProvider.mockDeleteManyFn,
            mockDeleteAllFn: mockProvider.mockDeleteAllFn,
            mockDescribeIndexStatsFn: mockProvider.mockDescribeIndexStatsFn
        }
    }
    
    public getPineconeMocks(): {
        mockDescribeIndexFn: Mock,
        mockCreateIndexFn: Mock,
        mockListIndexesFn: Mock,
        mockIndexFn: Mock
        } {
        return {
            mockDescribeIndexFn: mockProvider.mockDescribeIndexFn,
            mockCreateIndexFn: mockProvider.mockCreateIndexFn,
            mockListIndexesFn: mockProvider.mockListIndexesFn,
            mockIndexFn: mockProvider.mockIndexFn
        }
    }
    
    public getNamespaceMocks(): {
        mockDeleteNamespaceFn: Mock,
        mockListNamespacesFn: Mock
        } {
        return {
            mockDeleteNamespaceFn: mockProvider.mockDeleteNamespaceFn,
            mockListNamespacesFn: mockProvider.mockListNamespacesFn
        }
    }
}

// ==== CONVENIENCE EXPORTS ====
export const pineconeMocks = PineconeMockFactory.getInstance()

// Legacy export for backward compatibility
export const pineconeMockFactory = pineconeMocks

// Legacy module export (kept for backward compatibility)
export const mockPineconeModule = mockedPineconeModule

// ==== DEFAULT EXPORT ====
export default pineconeMocks 
```