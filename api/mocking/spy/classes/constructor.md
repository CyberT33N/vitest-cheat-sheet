# Vitest Constructor Spying - Definitive Anleitung

## ❗ Critical Rules

**VERWENDE NIEMALS** einfaches `vi.spyOn()` für Class Constructor Spying, da dies die 'new'-Semantik zerstört. **IMPLEMENTIERE STATTDESSEN** das `vi.mock()` mit `mockImplementation` Pattern, um Constructor-Aufrufe zu überwachen während die Original-Instanziierung erhalten bleibt.

**ZENTRALE PRINZIPIEN:**
- **'new'-Semantik erhalten**: Constructor muss als Constructor aufrufbar bleiben
- **Original-Verhalten**: Echte Instanzen mit echtem Verhalten erstellen
- **Spy-Funktionalität**: Constructor-Aufrufe und Parameter überwachen
- **Type Safety**: Vollständige TypeScript-Unterstützung
- **Mock Reset**: Saubere Trennung zwischen Tests

## 📚 **Das Problem: Warum einfacher Spy auf Klassenkonstruktoren nicht funktioniert**

<example type="invalid">
```typescript
import * as moduleAlias from 'path/to/MyClass.ts'

// ❌ Dieser Ansatz schlägt fehl:
const spyOnClass = vi.spyOn(moduleAlias, 'MyClass')

// Error: "Class constructor MyClass cannot be invoked without 'new'"
// Grund: vi.spyOn() ersetzt den Constructor durch eine normale Mock-Funktion
//        ohne 'new'-Semantik
```
</example>

**Warum es nicht funktioniert:**
- `vi.spyOn()` ohne `mockImplementation()` überschreibt die Klasse vollständig
- Die **'new'-Semantik** geht verloren
- Der Constructor kann nicht mehr als Constructor aufgerufen werden

## ✅ **Die Lösung: vi.mock() mit mockImplementation Pattern**

<example>
```typescript
// ==== IMPORTS ====
import { MyClass, type IMyClass } from 'path/to/MyClass.ts'

// ==== MOCKS ====
vi.mock('path/to/MyClass.ts', async () => {
    type MyClassModuleType = typeof import('path/to/MyClass.ts')
    const actual = await vi.importActual<MyClassModuleType>('path/to/MyClass.ts')

    const mockedMyClass = vi.fn().mockImplementation((
        ...args: readonly [...ConstructorParameters<typeof actual.MyClass>]
    ): IMyClass => {
        return new actual.MyClass(...args)
    })

    // Wichtig: Prototype beibehalten für Spying!
    mockedMyClass.prototype = actual.MyClass.prototype

    return {
        ...actual, // Exportiere alle anderen Exports
        // eslint-disable-next-line @typescript-eslint/naming-convention
        MyClass: mockedMyClass
    }
})

// ==== TESTS ====
describe('ServiceUnderTest', () => {
    // Mock-Instanz für Assertions
    const mockedMyClass = vi.mocked(MyClass)

    beforeEach(() => {
        // Reset mock zwischen Tests
        mockedMyClass.mockClear()
    })

    it('should call constructor with correct arguments', () => {
        // Arrange
        const testData = {
            id: 'test-123',
            name: 'Test User',
            email: 'test@example.com'
        }

        // Act
        const result = serviceUnderTest.createEntity(testData)

        // Assert - Prüfe Constructor-Aufruf
        expect(mockedMyClass).toHaveBeenCalledWith(
            testData.id,
            testData.name,
            testData.email,
            expect.any(Date), // Für dynamische Werte
            expect.stringMatching(/^[A-Z]{2}\d{4}$/), // Für Pattern-Matching
            expect.objectContaining({ status: 'active' }) // Für Object-Matching
        )
        
        // Assert - Prüfe einmaliger Aufruf
        expect(mockedMyClass).toHaveBeenCalledTimes(1)
        
        // Assert - Prüfe Ergebnis-Instanz
        expect(result).toBeInstanceOf(MyClass)
        expect(result.id).toBe(testData.id)
        expect(result.name).toBe(testData.name)
    })

    it('should handle multiple constructor calls', () => {
        // Act
        serviceUnderTest.createMultipleEntities([
            { id: '1', name: 'User 1' },
            { id: '2', name: 'User 2' }
        ])

        // Assert
        expect(mockedMyClass).toHaveBeenCalledTimes(2)
        expect(mockedMyClass).toHaveBeenNthCalledWith(1, '1', 'User 1')
        expect(mockedMyClass).toHaveBeenNthCalledWith(2, '2', 'User 2')
    })
})

// ==== ERWEITERTE TESTS MIT SPYING ====
describe('Positive Tests ✅', () => {
    let setInsuranceDataSpy: MockInstance

    beforeEach(() => {
        // Spy auf Instanz-Methoden über Prototype
        setInsuranceDataSpy = vi.spyOn(MyClass.prototype, 'setInsuranceData')
    })

    it('sollte Constructor und Methoden korrekt aufrufen', () => {
        // Arrange
        const testData = { id: 'test-123', name: 'Test User' }

        // Act
        const result = serviceUnderTest.createAndConfigureEntity(testData)

        // Assert - Constructor-Aufruf
        expect(mockedMyClass).toHaveBeenCalledWith(
            testData.id,
            testData.name,
            expect.any(String)
        )

        // Assert - Methoden-Aufruf über Spy
        expect(setInsuranceDataSpy).toHaveBeenCalledWith(
            expect.objectContaining({ type: 'insurance' })
        )

        // Assert - Ergebnis
        expect(result).toBeInstanceOf(MyClass)
        expect(result.id).toBe(testData.id)
    })
})
```
</example>

## ⚠️ **Häufige Fallstricke**

### **1. Endlosschleife vermeiden**
```typescript
// ❌ Endlosschleife:
return new MyClass(...args) // Ruft Mock auf → Endlosschleife

// ✅ Korrekt:
return new actual.MyClass(...args) // Ruft Original auf
```

### **2. Mock Reset nicht vergessen**
```typescript
beforeEach(() => {
    mockedClass.mockClear() // Reset call history
})
```

### **3. TypeScript-Imports**
```typescript
// ✅ Typ und Implementierung importieren
import { MyClass, type IMyClass } from 'path/to/MyClass.ts'

// ❌ Nur Typ importieren reicht nicht
import type { MyClass } from 'path/to/MyClass.ts'
```

## 🏆 **Best Practices**

1. **Mock am Datei-Anfang** nach Imports platzieren
2. **vi.importActual()** verwenden um Original zu erhalten
3. **ConstructorParameters<T>** für Typsicherheit
4. **mockedClass.mockClear()** in beforeEach()
5. **Sowohl Constructor-Aufruf als auch Ergebnis** testen
6. **Spezifische Matcher** für unterschiedliche Argument-Typen verwenden

## 🔧 **Implementation Guidelines**

### **Template für beliebige Klassen:**

```typescript
// 1. ✅ Mock Setup
vi.mock('path/to/YourClass.ts', async () => {
    type YourClassModuleType = typeof import('path/to/YourClass.ts')
    const actual = await vi.importActual<YourClassModuleType>('path/to/YourClass.ts')

    return {
        ...actual,
        YourClass: vi.fn().mockImplementation((
            ...args: readonly [...ConstructorParameters<typeof actual.YourClass>]
        ) => {
            return new actual.YourClass(...args)
        })
    }
})

// 2. ✅ Test Setup
const mockedYourClass = vi.mocked(YourClass)

beforeEach(() => {
    mockedYourClass.mockClear()
})

// 3. ✅ Assertions
expect(mockedYourClass).toHaveBeenCalledWith(expectedArgs)
expect(mockedYourClass).toHaveBeenCalledTimes(expectedCount)
expect(result).toBeInstanceOf(YourClass)
```

**💡 Dieses Pattern ist der Standard für Constructor Spying in Vitest und funktioniert zuverlässig für alle Klassen-Szenarien.**

## 🎯 **Anwendungsfälle**

- **Service-Klassen** die andere Services instanziieren
- **Factory-Pattern** mit dynamischer Klassenerstellung
- **Dependency Injection** mit Constructor-Parametern
- **Error-Handling** bei Constructor-Fehlern
- **Performance-Tests** mit Constructor-Aufrufen

Dieser Ansatz gewährleistet vollständige Kontrolle über Constructor-Aufrufe bei gleichzeitiger Erhaltung des Original-Verhaltens der Klassen.