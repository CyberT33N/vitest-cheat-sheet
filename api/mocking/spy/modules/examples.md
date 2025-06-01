
# Vitest Spy Pattern für externe Module - Referenzguide

### ✅ Korrekte Implementierung

```typescript
// 1. Import des gesamten Moduls (nicht destrukturiert)
import lodash from 'lodash'
import { vi, type MockInstance } from 'vitest'

// 2. Spy-Variable deklarieren
let chunkSpy: MockInstance

// 3. Spy im beforeEach erstellen
beforeEach(() => {
    // Spy direkt auf das importierte Modul-Objekt
    chunkSpy = vi.spyOn(lodash, 'chunk')
})

// 4. Assertions im Test
it('sollte chunk mit korrekten Parametern aufrufen', async () => {
    // ... Test-Code ...
    
    // Prüfe Spy-Aufrufe
    expect(chunkSpy).toHaveBeenCalledWith(expectedArray, expectedSize)
    expect(chunkSpy).toHaveBeenCalledTimes(1)
})
```

### ❌ Häufige Fehler

```typescript
// FALSCH: Destrukturierter Import + Objekt-Wrapper
import { chunk } from 'lodash'
chunkSpy = vi.spyOn({ chunk }, 'chunk') // ❌ Funktioniert nicht!

// FALSCH: Spy auf lokale Variable
import { chunk } from 'lodash'
chunkSpy = vi.spyOn(chunk) // ❌ TypeError!
```

## Warum funktioniert es so?

1. **Modul-Referenz**: `vi.spyOn(lodash, 'chunk')` überwacht die `chunk`-Eigenschaft des `lodash`-Objekts
2. **Gleiche Referenz**: Wenn der zu testende Code `lodash.chunk()` aufruft, wird derselbe Spy getriggert
3. **Keine Überschreibung**: Die originale Funktionalität bleibt erhalten, nur die Aufrufe werden überwacht

## Best Practices

- ✅ Importiere das gesamte Modul: `import lodash from 'lodash'`
- ✅ Verwende `vi.spyOn(moduleObject, 'functionName')`
- ✅ Deklariere Spy-Variablen mit korrektem Typ: `MockInstance`
- ✅ Erstelle Spies im `beforeEach()` für saubere Test-Isolation
- ❌ Vermeide Mocking von Utility-Libraries (folge der Regel: nur externe Dependencies mocken)

## Template für externe Module

```typescript
import externalModule from 'external-library'
import { vi, type MockInstance } from 'vitest'

describe('MyService', () => {
    let functionSpy: MockInstance
    
    beforeEach(() => {
        functionSpy = vi.spyOn(externalModule, 'targetFunction')
    })
    
    it('should call external function correctly', () => {
        // Test implementation
        expect(functionSpy).toHaveBeenCalledWith(expectedArgs)
    })
})
```
