
# 🧪 **AI-Agent Testing Rule: Default Parameter Testing mit Behavior-Driven Approach**

## ❗ Critical Rules

**NIEMALS** Default-Parameter durch direktes Spying auf die gleiche Funktion testen.

**STATTDESSEN** immer **BEHAVIOR-DRIVEN** testen durch Spying auf **Child-Methoden/Dependencies**.

## 📋 **Zusammenfassung der Problematik**

### **Ursprüngliche Situation**
```typescript
// Service-Methode mit Default-Parameter
export class FileProcessingService {
    async processFiles<T>(
        filePaths: readonly string[],
        filterFn: ((record: T) => boolean) | null | undefined,
        stopFn: ((record: T) => boolean) | null | undefined,
        filterOptions: ReadonlyDeep<FilterOptionsAdvanced> | null | undefined,
        stopAtFirstResult = true  // ← DEFAULT PARAMETER
    ): Promise<T[]> {
        // Hier wird eine interne Funktion aufgerufen
        return await this.readFromFiles(filePaths, filterFn, stopFn, filterOptions, stopAtFirstResult)
    }

    private async readFromFiles<T>(
        filePaths: readonly string[],
        filterFn: ((record: T) => boolean) | null | undefined,
        stopFn: ((record: T) => boolean) | null | undefined,
        filterOptions: ReadonlyDeep<FilterOptionsAdvanced> | null | undefined,
        stopAtFirstResult: boolean
    ): Promise<T[]> {
        // Implementation mit stopAtFirstResult Logik
        for (let i = filePaths.length - 1; i >= 0; i--) {
            const result = await this.processFile(filePaths[i], filterFn, stopFn, filterOptions)
            if (result.length > 0 && stopAtFirstResult) {
                return result // ← Hier wird der Default-Parameter verwendet
            }
        }
        return []
    }
}
```

### **Fehlerhafter Test-Ansatz**
```typescript
// ❌ FALSCH: Direktes Spy auf die Service-Methode selbst
describe('FileProcessingService - WRONG APPROACH', () => {
    it('should call with default parameter - FAILS', async () => {
        const service = new FileProcessingService()
        const spy = vi.spyOn(service, 'processFiles')

        await service.processFiles(testPaths, null, null, null)
        // stopAtFirstResult nicht übergeben

        // ❌ SCHEITERT: Spy kann Default-Parameter nicht korrekt erkennen
        expect(spy).toHaveBeenCalledWith(testPaths, null, null, null, true)
    })
})
```

---

## 🎯 **AI-Agent Testing Regel**

> ### **DEFAULT PARAMETER TESTING RULE**
> 
> **NIEMALS** Default-Parameter durch direktes Spying auf die gleiche Funktion testen.
> 
> **STATTDESSEN** immer **BEHAVIOR-DRIVEN** testen durch Spying auf **Child-Methoden/Dependencies**.

---

## 🛠️ **Behavior-Driven Code-Beispiel mit Vitest**

### **Problem-Pattern**

<example type="invalid">

```typescript
// ❌ ANTI-PATTERN: Funktioniert NICHT
describe('FileProcessingService - ANTI-PATTERN', () => {
    let service: FileProcessingService

    beforeEach(() => {
        service = new FileProcessingService()
    })

    it('❌ should NOT test default parameters this way', async () => {
        // ❌ Spy auf die Methode selbst - SCHEITERT
        const spy = vi.spyOn(service, 'processFiles')
        
        await service.processFiles(testPaths, null, null, null)
        
        // ❌ Default-Parameter wird nicht erkannt
        expect(spy).toHaveBeenCalledWith(testPaths, null, null, null, true)
    })
})
```

</example>

### **✅ Behavior-Driven Lösung: Child-Method Spying**

<example>

```typescript
// ✅ BEST PRACTICE: Spy auf Child-Methoden
describe('FileProcessingService - BEHAVIOR-DRIVEN SOLUTION', () => {
    let service: FileProcessingService

    beforeEach(() => {
        service = new FileProcessingService()
    })

    describe('Default Parameter Testing', () => {
        it('✅ should call readFromFiles with default stopAtFirstResult=true', async () => {
            // ════════════════════════════╡ 🧱 ARRANGE ╞════════════════════════════
            const testPaths = ['/test/file1.dbf', '/test/file2.dbf']
            
            // ✅ Spy auf die CHILD-METHODE, nicht auf die getestete Methode
            const readFromFilesSpy = vi.spyOn(service, 'readFromFiles')
            readFromFilesSpy.mockResolvedValue([])

            // ════════════════════════════╡ 🎬 ACT ╞════════════════════════════════
            // Parent-Methode OHNE stopAtFirstResult aufrufen
            await service.processFiles(
                testPaths,
                null,
                null,
                null
                // stopAtFirstResult nicht übergeben - sollte default true sein
            )

            // ════════════════════════════╡ 🎯 EXPECT ╞══════════════════════════════
            // ✅ Child-Methode wird mit Default-Wert aufgerufen
            expect(readFromFilesSpy).toHaveBeenCalledExactlyOnceWith(
                testPaths,
                null,
                null,
                null,
                true  // ← Hier ist der Default-Parameter sichtbar!
            )
        })

        it('✅ should call readFromFiles with explicit stopAtFirstResult=false', async () => {
            // ════════════════════════════╡ 🧱 ARRANGE ╞════════════════════════════
            const testPaths = ['/test/file1.dbf']
            const readFromFilesSpy = vi.spyOn(service, 'readFromFiles')
            readFromFilesSpy.mockResolvedValue([])

            // ════════════════════════════╡ 🎬 ACT ╞════════════════════════════════
            // Mit explizitem Parameter aufrufen
            await service.processFiles(
                testPaths,
                null,
                null,
                null,
                false  // ← Explizit false übergeben
            )

            // ════════════════════════════╡ 🎯 EXPECT ╞══════════════════════════════
            // ✅ Child-Methode wird mit explizitem Wert aufgerufen
            expect(readFromFilesSpy).toHaveBeenCalledExactlyOnceWith(
                testPaths,
                null,
                null,
                null,
                false  // ← Expliziter Parameter
            )
        })
    })

    describe('Behavior Verification', () => {
        it('✅ should stop at first result when using default (true)', async () => {
            // ════════════════════════════╡ 🧱 ARRANGE ╞════════════════════════════
            const testPaths = ['/test/file1.dbf', '/test/file2.dbf', '/test/file3.dbf']
            
            // Mock processFile um kontrollierten Test zu ermöglichen
            const processFileSpy = vi.spyOn(service, 'processFile')
            processFileSpy
                .mockResolvedValueOnce([]) // file1: leer
                .mockResolvedValueOnce([{ id: 1 }]) // file2: hat Daten
                .mockResolvedValueOnce([{ id: 2 }]) // file3: sollte nicht aufgerufen werden

            // ════════════════════════════╡ 🎬 ACT ╞════════════════════════════════
            const result = await service.processFiles(testPaths, null, null, null)
            // stopAtFirstResult=true (Default)

            // ════════════════════════════╡ 🎯 EXPECT ╞══════════════════════════════
            // ✅ Behavior: Stoppt nach erstem Ergebnis
            expect(result).toEqual([{ id: 1 }])
            
            // ✅ Nur 2 Dateien wurden verarbeitet (stoppt nach erstem Ergebnis)
            expect(processFileSpy).toHaveBeenCalledTimes(2)
            expect(processFileSpy).toHaveBeenNthCalledWith(1, '/test/file3.dbf', null, null, null)
            expect(processFileSpy).toHaveBeenNthCalledWith(2, '/test/file2.dbf', null, null, null)
            // file1 wurde NICHT aufgerufen wegen stopAtFirstResult=true
        })

        it('✅ should process all files when stopAtFirstResult=false', async () => {
            // ════════════════════════════╡ 🧱 ARRANGE ╞════════════════════════════
            const testPaths = ['/test/file1.dbf', '/test/file2.dbf', '/test/file3.dbf']
            
            const processFileSpy = vi.spyOn(service, 'processFile')
            processFileSpy
                .mockResolvedValueOnce([]) // file1: leer
                .mockResolvedValueOnce([{ id: 1 }]) // file2: hat Daten
                .mockResolvedValueOnce([{ id: 2 }]) // file3: hat auch Daten

            // ════════════════════════════╡ 🎬 ACT ╞════════════════════════════════
            const result = await service.processFiles(
                testPaths,
                null,
                null,
                null,
                false  // ← Explizit false: Alle Dateien verarbeiten
            )

            // ════════════════════════════╡ 🎯 EXPECT ╞══════════════════════════════
            // ✅ Behavior: Verarbeitet alle Dateien
            expect(result).toEqual([{ id: 1 }, { id: 2 }])
            
            // ✅ Alle 3 Dateien wurden verarbeitet
            expect(processFileSpy).toHaveBeenCalledTimes(3)
        })
    })
})
```

</example>
