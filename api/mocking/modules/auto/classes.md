


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
        