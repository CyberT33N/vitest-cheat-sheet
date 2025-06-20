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
// ✅ KORREKT - Neue modulare Struktur
export class PineconeMockFactory {
    // Public Mock-Funktionen - direkt zugänglich, keine unbound methods
    public mockNamespaceFn: MockedFunction<(...args: readonly unknown[]) => unknown>
    public mockUpsertFn: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    // ... weitere Mock-Funktionen
    
    // Mock-Instanzen
    public mockPineconeInstance: Pinecone
    public mockIndexInstance: Index
    public mockPinecone: PineconeModule | undefined
    
    public constructor() {
        // Initialisierung aller Mock-Funktionen
        this.mockUpsertFn = vi.fn()
        this.mockQueryFn = vi.fn()
        // ...
        
        // Erstellung der Mock-Instanzen
        this.mockPineconeInstance = this._createMockPineconeClient()
        this.mockIndexInstance = this._createMockIndexObject()
        
        // Konfiguration der Rückgabewerte
        this.mockNamespaceFn.mockReturnValue(this.mockNamespaceInstance)
        this.mockIndexFn.mockReturnValue(this.mockIndexInstance)
    }
    
    public resetAllMocks(): void {
        // Zentrale Reset-Funktion für alle Mocks
    }
    
    private _createMockIndexObject(): Index { /* ... */ }
    private _createMockPineconeClient(): Pinecone { /* ... */ }
}

// Globale Factory-Instanz
export const pineconeMockFactory = new PineconeMockFactory()

// Mock-Modul Export-Funktion
export const mockPineconeModule = async(): Promise<PineconeModule> => {
    const original = await vi.importActual<PineconeModule>('@pinecone-database/pinecone')
    
    const mockedModule = {
        ...original,
        Pinecone: vi.fn().mockImplementation(() => pineconeMockFactory.mockPineconeInstance),
    }
    
    pineconeMockFactory.mockPinecone = mockedModule
    return mockedModule
}
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