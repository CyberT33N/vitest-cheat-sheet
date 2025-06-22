## **🔄 Runtime Module Mocking: `vi.doMock()` vs `vi.importMock()`**

### **📋 Kurze Zusammenfassung:**

| Aspekt | `vi.doMock()` | `vi.importMock()` |
|--------|---------------|-------------------|
| **Timing** | Runtime, dann re-import nötig | Runtime, import + mock in einem |
| **Syntax** | Zwei Schritte | Ein Schritt |
| **Flexibilität** | ✅ Sehr hoch | ✅ Sehr hoch |
| **Performance** | Langsamer (re-import) | Schneller (direkter import) |
| **Cleanup** | `vi.doUnmock()` | Automatisch pro Test |

---


### **3️⃣ `vi.importMock()` - Erweiterte Beispiele**

```typescript
describe('Advanced vi.importMock() patterns', () => {
    it('sollte spezifische Methoden mocken', async () => {
        const mockGoogleGenAI = await vi.importMock<typeof import('@google/genai')>('@google/genai')
        
        // Direkter Zugriff auf gemockte Funktionen
        const mockEmbedContent = vi.fn().mockResolvedValue({
            embeddings: [{ values: [0.1, 0.2, 0.3] }]
        })
        
        mockGoogleGenAI.GoogleGenAI.mockImplementation(() => ({
            models: { embedContent: mockEmbedContent }
        }))

        const { GoogleEmbeddingService } = await import('@/services/embedding/google/embedding-service.ts')
        const service = new GoogleEmbeddingService()
        
        await service.generateEmbeddings(['test'], { taskType: EGeminiTaskType.retrievalQuery })
        
        // Direkter Zugriff auf Mock für Assertions
        expect(mockEmbedContent).toHaveBeenCalledWith(expect.objectContaining({
            contents: [{ role: 'user', parts: [{ text: 'test' }] }]
        }))
    })

    it('sollte conditional mocking unterstützen', async () => {
        const mockGoogleGenAI = await vi.importMock<typeof import('@google/genai')>('@google/genai')
        
        let callCount = 0
        mockGoogleGenAI.GoogleGenAI.mockImplementation(() => ({
            models: {
                embedContent: vi.fn().mockImplementation(async () => {
                    callCount++
                    if (callCount === 1) {
                        return { embeddings: [{ values: [0.1, 0.2] }] }
                    } else {
                        throw new Error('Rate limit')
                    }
                })
            }
        }))

        const { GoogleEmbeddingService } = await import('@/services/embedding/google/embedding-service.ts')
        const service = new GoogleEmbeddingService()
        
        // Erster Call erfolgreich
        const result1 = await service.generateEmbeddings(['test1'], { taskType: EGeminiTaskType.retrievalQuery })
        expect(result1).toEqual([[0.1, 0.2]])
        
        // Zweiter Call fehlschlägt
        await expect(service.generateEmbeddings(['test2'], { taskType: EGeminiTaskType.retrievalQuery }))
            .rejects.toThrow('Rate limit')
    })

    it('sollte partial mocking ermöglichen', async () => {
        const mockGoogleGenAI = await vi.importMock<typeof import('@google/genai')>('@google/genai')
        
        // Nur spezifische Teile mocken, Rest bleibt original
        mockGoogleGenAI.GoogleGenAI.mockImplementation(() => ({
            models: {
                embedContent: vi.fn().mockResolvedValue({
                    embeddings: [{ values: [0.1, 0.2, 0.3] }]
                }),
                // Andere Methoden könnten original bleiben oder auch gemockt werden
                generateContent: vi.fn() // Falls vorhanden
            }
        }))

        const { GoogleEmbeddingService } = await import('@/services/embedding/google/embedding-service.ts')
        const service = new GoogleEmbeddingService()
        
        const result = await service.generateEmbeddings(['test'], { taskType: EGeminiTaskType.retrievalQuery })
        expect(result).toEqual([[0.1, 0.2, 0.3]])
    })
})
```

---

## **🎯 Kernunterschiede:**

### **`vi.doMock()` - Klassischer Ansatz**
```typescript
// Schritt 1: Mock definieren
vi.doMock('@google/genai', () => ({ /* mock */ }))

// Schritt 2: Importieren
const { Service } = await import('./service')

// Schritt 3: Cleanup (manuell)
vi.doUnmock('@google/genai')
```

### **`vi.importMock()` - Moderner Ansatz**
```typescript
// Alles in einem: Import + Mock + Auto-Cleanup
const mockModule = await vi.importMock<typeof import('@google/genai')>('@google/genai')

// Direkte Mock-Konfiguration
mockModule.GoogleGenAI.mockImplementation(() => ({ /* mock */ }))

// Service normal importieren
const { Service } = await import('./service')
// Automatisches Cleanup nach Test!
```

---

## **🏆 Empfehlung:**

**Für neue Tests:** Verwende `vi.importMock()` - es ist sauberer, moderner und hat automatisches Cleanup.

**Für bestehende Tests:** `vi.doMock()` funktioniert weiterhin perfekt.

**`vi.importMock()` ist die Evolution von `vi.doMock()` - weniger Boilerplate, mehr Typsicherheit!** 🚀
