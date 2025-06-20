
## **🎯 Praktische Beispiele zum Kopieren:**

### **1️⃣ `vi.doMock()` - Zwei-Schritt Ansatz**

```typescript
describe('Dynamic Mocking with vi.doMock()', () => {
    afterEach(() => {
        vi.doUnmock('@google/genai') // Cleanup nötig
    })

    it('sollte verschiedene Mock-Verhalten pro Test haben', async () => {
        // Schritt 1: Mock definieren
        vi.doMock('@google/genai', () => ({
            GoogleGenAI: vi.fn().mockImplementation(() => ({
                models: {
                    embedContent: vi.fn().mockResolvedValue({
                        embeddings: [{ values: [0.1, 0.2, 0.3] }]
                    })
                }
            }))
        }))

        // Schritt 2: Service NACH Mock importieren (wichtig!)
        const { GoogleEmbeddingService } = await import('@/services/embedding/google/embedding-service.ts')
        const service = new GoogleEmbeddingService()
        
        const result = await service.generateEmbeddings(['test'], { 
            taskType: EGeminiTaskType.retrievalQuery 
        })
        
        expect(result).toEqual([[0.1, 0.2, 0.3]])
    })
})
```

### **2️⃣ `vi.importMock()` - Ein-Schritt Ansatz**

```typescript
describe('Dynamic Mocking with vi.importMock()', () => {
    // Kein afterEach nötig - automatisches Cleanup!

    it('sollte verschiedene Mock-Verhalten pro Test haben', async () => {
        // Ein Schritt: Import + Mock gleichzeitig
        const mockGoogleGenAI = await vi.importMock<typeof import('@google/genai')>('@google/genai')
        
        // Mock konfigurieren
        mockGoogleGenAI.GoogleGenAI.mockImplementation(() => ({
            models: {
                embedContent: vi.fn().mockResolvedValue({
                    embeddings: [{ values: [0.1, 0.2, 0.3] }]
                })
            }
        }))

        // Service normal importieren
        const { GoogleEmbeddingService } = await import('@/services/embedding/google/embedding-service.ts')
        const service = new GoogleEmbeddingService()
        
        const result = await service.generateEmbeddings(['test'], { 
            taskType: EGeminiTaskType.retrievalQuery 
        })
        
        expect(result).toEqual([[0.1, 0.2, 0.3]])
    })

    it('sollte anderen Mock für anderen Test verwenden', async () => {
        // Komplett neuer Mock - automatisch isoliert
        const mockGoogleGenAI = await vi.importMock<typeof import('@google/genai')>('@google/genai')
        
        mockGoogleGenAI.GoogleGenAI.mockImplementation(() => ({
            models: {
                embedContent: vi.fn().mockRejectedValue(new Error('API Error'))
            }
        }))

        const { GoogleEmbeddingService } = await import('@/services/embedding/google/embedding-service.ts')
        const service = new GoogleEmbeddingService()
        
        await expect(service.generateEmbeddings(['test'], { 
            taskType: EGeminiTaskType.retrievalQuery 
        })).rejects.toThrow('API Error')
    })
})
```