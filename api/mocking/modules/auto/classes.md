# Auto Mocking

## Classes
- Die folgenden Beispiele sind kein vollständig vollständiges **Automocking**, da die neu erstellte **Klasseninstanz**, die in anderen Dateien erstellt wird, mit einem **Mock** überschrieben werden **MUSS**.

<details><summary>Click to expand..</summary>

### Zusammenfassung: Mocking von Klassen (insb. von externen Modulen) in Vitest

Wenn du Klassen mocken musst, insbesondere solche, die von externen Modulen exportiert werden (z.B. SDK-Clients), unterscheidet sich der Ansatz vom Mocking einfacher Objekte oder Funktionen. Das Hauptziel ist oft, den **Konstruktor** der Klasse zu kontrollieren und/oder **Methoden von Instanzen** dieser Klasse zu mocken.

Es gibt verschiedene Wege, dies zu erreichen. Hier sind zwei gängige Ansätze, die in den Beispielen gezeigt werden:

1.  **Hoisted Mock Factory mit `mockObject` und `vi.importActual`**:
    *   Dieser Ansatz ist sehr explizit und nutzt `vi.hoisted()` um eine Factory zu erstellen, die das Mock-Setup vor allen anderen Modul-Imports durchführt.
    *   `vi.importActual` lädt das originale Modul.
    *   `mockObject` (von `vitest/mocker`) erstellt ein gemocktes Objekt des Originals.
    *   Entscheidend ist, den **Konstruktor der Klasse** innerhalb des gemockten Moduls zu überschreiben (z.B. `gemocktesModul.KlassenName = vi.fn().mockImplementation(() => mockInstanz)`), sodass er eine von dir definierte **Mock-Instanz** zurückgibt.
    *   Diese Mock-Instanz enthält dann die gemockten Methoden (z.B. `methodenName: vi.fn()`), die du in deinen Tests steuern und überwachen kannst.
    *   **Vorteil:** Klare Struktur, einfacher Zugriff auf die Mock-Instanz und ihre Methoden im Test.
    *   **Nachteil:** Etwas mehr Boilerplate durch die Factory.

2.  **Direkter Modul-Mock mit `vi.fn().mockImplementation()` für die Klasse**:
    *   Hier wird das Modul direkt innerhalb von `vi.mock('modul-pfad', () => { ... })` gemockt.
    *   Die exportierte Klasse selbst wird durch ein `vi.fn()` ersetzt.
    *   Die `.mockImplementation(() => { return { /* gemockte Instanzmethoden */ }; })` dieser Funktion gibt dann ein Objekt zurück, das eine Instanz der Klasse simuliert. Die Methoden dieses Objekts sind wiederum `vi.fn()`.
    *   **Vorteil:** Kompakter, da keine separate Factory-Struktur nötig ist.
    *   **Nachteil:** Der Zugriff auf die *gemockten Methoden der Instanz* im Test-Setup kann etwas umständlicher sein, oft über `gemockteKlasse.mock.results[index].value.methodenName`, da jede Instanziierung der Klasse (z.B. durch `new GemockteKlasse()`) ein neues "result" im Mock-Objekt der Klasse erzeugt.

Beide Ansätze ermöglichen es dir, das Verhalten von Klasseninstanzen präzise für deine Unit-Tests zu steuern.
















<br><br>

---

<br><br>


### Beispiel 1: `__mocks__` **PREFERRED**


# Vitest Mock Refaktorierung: Von Hoisted zu Modularer Struktur

## Übersicht

Diese Dokumentation beschreibt die Refaktorierung der Pinecone-Service-Tests von einem komplexen `vi.hoisted()` Ansatz zu einer sauberen, modularen Mock-Struktur.

## Problem: Ursprünglicher Hoisted-Ansatz

### Probleme des alten Ansatzes

1. **Unbound Method Errors**: Direkte Referenzen auf Mock-Methoden verursachten ESLint-Fehler
2. **Komplexität**: Über 200 Zeilen Mock-Code direkt in der Test-Datei
3. **Wartbarkeit**: Schwer zu verstehen und zu erweitern
4. **Wiederverwendbarkeit**: Mock-Code war nicht zwischen Tests teilbar

### Alter Code-Struktur

```typescript
// ❌ PROBLEMATISCH - Alter Ansatz
const mockFactory = vi.hoisted(() => {
    let mockedPineconeModule: MockedPineconeModule
    let mockPineconeInstance: Pinecone
    // ... 200+ Zeilen Mock-Code
    
    const createMockIndexObject = (): Index => {
        return {
            namespace: mockNamespaceFn,  // ❌ Unbound method
            upsert: mockUpsertFn,        // ❌ Unbound method
            // ... weitere Mock-Methoden
        }
    }
    
    return {
        getMockedPineconeModule: () => mockedPineconeModule,
        getMockIndexInstance: () => mockIndexInstance,
        // ... viele Getter-Funktionen
    }
})

vi.mock('@pinecone-database/pinecone', async() => {
    const module = await mockFactory.createAndStoreMockedModule()
    return module
})
```

## Lösung: Modulare Mock-Struktur

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

**Lösung**: Inline-Import in der Mock-Definition:
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

---

<br><br>


### Beispiel 2: Hoisted Mock Factory mit `mockObject`


<details><summary>Click to expand..</summary>

Dieser Ansatz ist nützlich, um eine klare Struktur für das Mocking eines externen Moduls und seiner Klassen zu schaffen. Die Factory stellt sicher, dass die Mocks korrekt initialisiert werden, bevor der Testcode ausgeführt wird.

```typescript
// ==== Imports ====
import { describe, it, expect, vi, beforeEach, type MockedObject, type MockedFunction } from 'vitest'
import { mockObject } from 'vitest/mocker'

import env from '@/env.js'
import {
    EGeminiTaskType, type IEmbeddingServiceConfig
} from '@/services/embedding/embedding-types.js'
import { GoogleEmbeddingService } from '@/services/embedding/google/embedding-service.ts'
import { NonEmptyArrayReadOnly } from '@/utils/types.ts'

// Import helper functions
import {
    TEST_DATA,
    testConstructorDefaults,
    testConstructorWithConfig,
    testGetDefaultDimension,
    createRetrievalQueryConfig,
    createRetrievalDocumentConfig,
    createCustomDimensionConfig,
    testImmutableResults,
    testEmptyValueHandling,
    createStandardService,
    createServiceWithConfig
} from '@test/common/src/services/embedding/google/embedding-service-helpers.ts'

// Type für das gesamte Google GenAI-Modul
type GoogleGenAIModule = typeof import('@google/genai')
// Type für eine gemockte Version des Google GenAI-Moduls
type MockedGoogleGenAIModule = MockedObject<GoogleGenAIModule>

// Type für die Mock-Instanz (die von new GoogleGenAI() zurückgegeben wird)
interface IMockGoogleGenAIInstance {
    models: {
        embedContent: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    }
}

// ==== Mocks ====
const mockFactory = vi.hoisted(() => {
    let mockedGoogleGenAIModule: MockedGoogleGenAIModule
    let mockGoogleGenAIInstance: IMockGoogleGenAIInstance
    
    // Functional approach: Factory function instead of setter
    const createAndStoreMockedModule = async(): Promise<MockedGoogleGenAIModule> => {
        // 1. Original Modul laden
        const original = await vi.importActual<GoogleGenAIModule>('@google/genai')
        
        // 2. Mock-Instanz erstellen, die der Konstruktor zurückgeben soll
        // Diese Instanz enthält die Methoden, die wir mocken wollen.
        mockGoogleGenAIInstance = {
            models: {
                embedContent: vi.fn() // Dies ist die Methode der Instanz
            }
        }
        
        // 3. Das gesamte Modul mocken
        const module = mockObject(
            {
                type: 'automock', // Versucht, alles automatisch zu mocken
                spyOn: vi.spyOn,
                globalConstructors: { // Wichtig für mockObject
                    Object,
                    Function,
                    RegExp,
                    Array,
                    Map
                }
            },
            original
        ) as MockedGoogleGenAIModule
        
        // 4. Den Konstruktor der Klasse im gemockten Modul explizit mocken,
        //    sodass er unsere mockGoogleGenAIInstance zurückgibt.
        module.GoogleGenAI = vi.fn().mockImplementation(() => mockGoogleGenAIInstance)
        
        mockedGoogleGenAIModule = module
        return module
    }
    
    return {
        getMockedGoogleGenAIModule: (): MockedGoogleGenAIModule => mockedGoogleGenAIModule,
        getMockGoogleGenAIInstance: (): IMockGoogleGenAIInstance => mockGoogleGenAIInstance,
        createAndStoreMockedModule
    }
})

// Das Modul '@google/genai' wird gemockt.
// Wenn es importiert wird, wird stattdessen das Ergebnis dieser Funktion verwendet.
vi.mock('@google/genai', async(): Promise<MockedGoogleGenAIModule> => {
    const module = await mockFactory.createAndStoreMockedModule()
    return module
})

// ==== Tests ====
describe('GoogleEmbeddingService() - Unit Tests with Mocks (Hoisted Factory)', () => {
    let service: GoogleEmbeddingService
    let mockedGoogleGenAIConstructor: MockedFunction<any> // Referenz zum gemockten Konstruktor
    let mockGoogleGenAIInstance: IMockGoogleGenAIInstance // Referenz zur Mock-Instanz

    beforeEach(() => {
        // Zugriff auf das gemockte Modul und die Mock-Instanz über die Factory
        const fullMockedModule = mockFactory.getMockedGoogleGenAIModule()
        mockedGoogleGenAIConstructor = fullMockedModule.GoogleGenAI as MockedFunction<any>
        mockGoogleGenAIInstance = mockFactory.getMockGoogleGenAIInstance()
        
        vi.stubEnv('GEMINI_API_KEY', env.GEMINI_API_KEY)
        
        // Service wird NACH dem Setup der Mocks erstellt
        service = createStandardService() // Dies ruft `new GoogleGenAI()` intern auf
    })

    describe('Constructor', () => {
        it('sollte erfolgreich initialisiert werden mit default values', () => {
            testConstructorDefaults(service)

            // Überprüfen, ob der gemockte Konstruktor aufgerufen wurde
            expect(mockedGoogleGenAIConstructor).toHaveBeenCalledWith(expect.objectContaining({
                apiKey: env.GEMINI_API_KEY
            }))
        })

        // ... weitere Konstruktor-Tests
    })

    describe('generateEmbeddings()', () => {
        describe('Single Text', () => {
            it('sollte ein einzelnes Embedding für RETRIEVAL_QUERY generieren', async() => {
                // Setup der Mock-Methode auf der Mock-Instanz
                mockGoogleGenAIInstance.models.embedContent.mockResolvedValueOnce({
                    embeddings: [{ values: TEST_DATA.embedding1 }]
                })

                const config = createRetrievalQueryConfig(TEST_DATA.title)
                const embeddings = await service.generateEmbeddings([TEST_DATA.text], config)

                expect(embeddings).toEqual([TEST_DATA.embedding1])
                // Überprüfen, ob die Methode der Mock-Instanz aufgerufen wurde
                expect(mockGoogleGenAIInstance.models.embedContent).toHaveBeenCalledWith(expect.objectContaining({
                    model: env.GEMINI_DEFAULT_EMBEDDING_MODEL,
                    // ... weitere Erwartungen
                }))
            })
            // ... weitere generateEmbeddings Tests
        })
    })
    // ... Rest der Tests
})
```

</details>












<br><br>


---

<br><br>



### Beispiel 2: `vi.mock` + `vi.mocked().mockImplementation()`

<details><summary>Click to expand..</summary>

```typescript
// ==== Imports ====
import { describe, it, expect, vi, beforeEach, type MockedFunction } from 'vitest';
import { GoogleGenAI } from '@google/genai'; // Angenommen, dies ist der Original-Import
import { GoogleEmbeddingService } from '@/services/embedding/google/embedding-service.ts'; // Dein Service
import env from '@/env.js';

// Dummy-Funktionen/Daten für das Beispiel
const createStandardService = () => new GoogleEmbeddingService();
const testConstructorDefaults = (service: any) => { /* ... */ };

// ==== Mocks ====
// 1. Mocke das gesamte Modul.
// GoogleGenAI (wenn importiert) wird dadurch bereits zu einer Mock-Funktion.
vi.mock('@google/genai');

// 2. Erstelle eine Mock-Instanz mit den benötigten Methoden
let mockEmbedContent: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>

/ 3. Überschreibe die Implementierung des GoogleGenAI-Konstruktors
// eslint-disable-next-line @typescript-eslint/no-explicit-any
vi.mocked(GoogleGenAI).mockImplementation((): any => {
    // Diese Funktion wird jedes Mal ausgeführt, wenn `new GoogleGenAI()` aufgerufen wird
    mockEmbedContent = vi.fn().mockImplementation(() => {
        return {
            embeddings: [{ values: TEST_DATA.embedding1 }]
        }
    })
    
    return {
        models: {
            embedContent: mockEmbedContent
        }
    }
})


// ==== Tests ====
describe('GoogleEmbeddingService() - Unit Tests with Mocks (vi.mocked().mockImplementation())', () => {
    let service: GoogleEmbeddingService;
    // Um auf die mockEmbedContent-Funktion der *letzten* Instanz zuzugreifen:
    let lastMockInstanceEmbedContent: MockedFunction<any>;

    beforeEach(async() => {
        service = createStandardService(); // Ruft intern `new GoogleGenAI()` auf
    });

    describe('Constructor', () => {
        it('sollte erfolgreich initialisiert werden mit default values', async() => {
            testConstructorDefaults(service); // Führt Assertions auf dem Service-Objekt aus

            // Überprüfe, ob der GoogleGenAI Konstruktor-Mock korrekt aufgerufen wurde
            expect(GoogleGenAI).toHaveBeenCalledTimes(1); // Sicherstellen, dass er nur einmal im beforeEach aufgerufen wurde
            expect(GoogleGenAI).toHaveBeenCalledWith(expect.objectContaining({
                apiKey: env.GEMINI_API_KEY
            }));
        });
    });
});
```









<br><br>

---

<br><br>

### Beispiel 3: Direkter Modul-Mock mit `vi.fn().mockImplementation()` für die Klasse

Dieser Ansatz ist oft kompakter. Der `vi.mock`-Callback gibt direkt ein Objekt zurück, das die exportierte Klasse als `vi.fn()` enthält, deren `mockImplementation` eine simulierte Instanz mit gemockten Methoden zurückgibt.

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
import { describe, it, expect, vi, beforeEach, type MockedFunction } from 'vitest'

import env from '@/env.js'
import {
    EGeminiTaskType, type IEmbeddingServiceConfig
} from '@/services/embedding/embedding-types.js'
import { GoogleEmbeddingService } from '@/services/embedding/google/embedding-service.ts'
// ... weitere Imports

// Import helper functions
import {
    TEST_DATA,
    testConstructorDefaults,
    // ... weitere Helfer
    createStandardService,
    createServiceWithConfig
} from '@test/common/src/services/embedding/google/embedding-service-helpers.ts'

// Type für die Mock-Instanz, die der gemockte Konstruktor zurückgibt
interface IMockGoogleGenAIInstance {
    models: {
        embedContent: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>>
    }
}

// ==== Mocks ====
// Mock nur das @google/genai Modul
vi.mock('@google/genai', () => {
    // Dies ist die Mock-Funktion für die Methode der *Instanz*
    const mockEmbedContentMethod = vi.fn()
    
    // Die Klasse GoogleGenAI wird durch ein vi.fn() ersetzt.
    // Ihre mockImplementation gibt ein Objekt zurück, das eine Instanz simuliert.
    const MockGoogleGenAIClass = vi.fn().mockImplementation(() => ({
        models: { // Die Struktur der echten Instanz nachbilden
            embedContent: mockEmbedContentMethod // Die gemockte Methode
        }
    }))
    
    return {
        GoogleGenAI: MockGoogleGenAIClass, // Exportiere die gemockte Klasse
        // Falls andere Exporte des Moduls benötigt werden, hier ebenfalls mocken:
        // EmbedContentParameters: vi.fn() 
    }
})

// ==== Tests ====
describe('GoogleEmbeddingService() - Unit Tests with Mocks (Direct Mock)', () => {
    let service: GoogleEmbeddingService
    let MockedGoogleGenAI: MockedFunction<any> // Typ für die gemockte Klasse/Konstruktor
    let mockEmbedContentOnInstance: MockedFunction<(...args: readonly unknown[]) => Promise<unknown>> // Typ für die gemockte Instanzmethode

    beforeEach(async() => {
        // Hole das gemockte Modul und die darin enthaltene gemockte Klasse
        // Wichtig: Das importierte `GoogleGenAI` ist bereits die Mock-Funktion von oben.
        const { GoogleGenAI } = await import('@google/genai')
        MockedGoogleGenAI = vi.mocked(GoogleGenAI) // `vi.mocked` für Typsicherheit und Zugriff auf Mock-Eigenschaften
        
        vi.stubEnv('GEMINI_API_KEY', env.GEMINI_API_KEY)
        
        // Erstelle Service (dies ruft intern `new GoogleGenAI()` auf, was unsere Mock-Implementierung auslöst)
        service = createStandardService() 
        
        // Zugriff auf die Mock-Instanz-Methoden:
        // Jedes Mal, wenn `new GoogleGenAI()` (also unser MockedGoogleGenAI) aufgerufen wird,
        // wird ein neues Ergebnis in `MockedGoogleGenAI.mock.results` gespeichert.
        // Das `.value` dieses Ergebnisses ist das, was die `mockImplementation` zurückgegeben hat
        // (also unser Objekt mit `{ models: { embedContent: mockEmbedContentMethod } }`).
        // Wir greifen auf die *zuletzt erstellte* Instanz zu.
        const lastMockInstanceDetails = MockedGoogleGenAI.mock.results[MockedGoogleGenAI.mock.results.length - 1]
        if (lastMockInstanceDetails && lastMockInstanceDetails.type === 'return') {
            const mockInstance = lastMockInstanceDetails.value as IMockGoogleGenAIInstance
            mockEmbedContentOnInstance = mockInstance.models.embedContent
        } else {
            // Fallback oder Fehlerbehandlung, falls die Instanz nicht wie erwartet erstellt wurde.
            // Für dieses Beispiel gehen wir davon aus, dass es immer klappt.
            // In einem realen Szenario könnte hier ein Fehler geworfen oder ein Default gesetzt werden.
            // Für den Test:
            if (!mockEmbedContentOnInstance && MockedGoogleGenAI.mock.calls.length > 0) {
                // Dies passiert, wenn MockedGoogleGenAI aufgerufen wurde, aber vielleicht kein `createStandardService`
                // das letzte Ergebnis produzierte, oder wenn die Struktur anders ist.
                // Im Normalfall sollte der obige `if`-Block greifen.
                // Für eine robustere Lösung könnte man die `mockEmbedContentMethod` direkt speichern
                // und wiederverwenden, die im äußeren Scope von `vi.mock` definiert wurde.
                // Dann wäre dieser Zugriff über `mock.results` nicht nötig.
                // Für dieses Beispiel bleiben wir aber bei der Logik des ursprünglichen Codes.
            }
        }
        
        // Reset nur die spezifischen Mocks, die wir kontrollieren wollen
        if (mockEmbedContentOnInstance) {
            mockEmbedContentOnInstance.mockReset()
        }
        MockedGoogleGenAI.mockClear() // Löscht Aufrufinformationen des Konstruktors
    })

    describe('Constructor', () => {
        it('sollte erfolgreich initialisiert werden mit default values', async() => {
            testConstructorDefaults(service)

            // Überprüfen, ob der gemockte Konstruktor (MockedGoogleGenAI) aufgerufen wurde
            expect(MockedGoogleGenAI).toHaveBeenCalledWith(expect.objectContaining({
                apiKey: env.GEMINI_API_KEY
            }))
        })

        // ... weitere Konstruktor-Tests
    })

    describe('generateEmbeddings()', () => {
        describe('Single Text', () => {
            it('sollte ein einzelnes Embedding für RETRIEVAL_QUERY generieren', async() => {
                // Stelle sicher, dass mockEmbedContentOnInstance initialisiert ist
                if (!mockEmbedContentOnInstance) throw new Error("mockEmbedContentOnInstance not initialized");

                // Setup der Mock-Methode
                mockEmbedContentOnInstance.mockResolvedValueOnce({
                    embeddings: [{ values: TEST_DATA.embedding1 }]
                })

                const config = createRetrievalQueryConfig(TEST_DATA.title)
                const embeddings = await service.generateEmbeddings([TEST_DATA.text], config)

                expect(embeddings).toEqual([TEST_DATA.embedding1])
                // Überprüfen, ob die gemockte Instanzmethode aufgerufen wurde
                expect(mockEmbedContentOnInstance).toHaveBeenCalledWith(expect.objectContaining({
                    model: env.GEMINI_DEFAULT_EMBEDDING_MODEL,
                    // ... weitere Erwartungen
                }))
            })
            // ... weitere generateEmbeddings Tests
        })
    })
    // ... Rest der Tests
})
```

</details>
        
</details>
        