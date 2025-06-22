
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