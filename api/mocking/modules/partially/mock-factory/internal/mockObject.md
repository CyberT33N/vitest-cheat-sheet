# Mock Factory 

## mockObject

### Beispiel: Hoisted Mock Factory mit `mockObject`


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





