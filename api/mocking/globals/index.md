# Mockings

## Globals

### Sync

<details><summary>Click to expand..</summary>

# Option 1 `stubGlobal` + `mockImplementation`

service.ts:
```typescript
/**
 * Converts a string to an integer.
 * @param value - The string.
 * @returns The integer.
 */
private _toInt(value: string): number {
    try {
        const num = parseInt(value)
        const result = (!num || isNaN(num)) ? 0 : num
        return result
    } catch (e) {
        Logger.error('Error in _toInt: ' + JSON.stringify(e, null, 2))
        return 0
    }
}
```

test.ts:
```typescript
describe('Negative Tests ❌', () => {
    let loggerErrorSpy: MockInstance

    beforeEach(() => {
        loggerErrorSpy = vi.spyOn(Logger, 'error')

        vi.stubGlobal('parseInt', vi.fn().mockImplementation(() => {
            throw new Error('Parse error')
        }))
    })

    afterEach(() => {
        vi.unstubAllGlobals()
    })

    it.only('sollte 0 zurückgeben und Fehler loggen bei Exception', () => {
        const result = context.service['_toInt']('123')

        expect(result).toBe(0)
        expect(loggerErrorSpy).toHaveBeenCalledWith(
            expect.stringContaining('Error in _toInt:')
        )
    })
})
```
- In manchen Fällen bringt es nichts, dass wir nur in der **Vitest-Config** das **unstableGlobals** auf **true** setzen, weil nachher beim Aufräumen gemockte Methoden aufgerufen werden. Daher **MÜSSEN** wir sicherstellen, dass wir nach der Testausführung es im **afterEach** zurücksetzen.

vitest.config.ts:
```
/*
Will call vi.unstubAllGlobals before each test.
> Entfernt gestubbte globale Objekte, z.B. `globalThis.fetch = vi.fn()`.

🧠 Wenn nicht global gesetzt – selbst aufräumen:

``ts
afterEach(() => {
  vi.unstubAllGlobals(); // Global-Stubs wie fetch, window.alert etc.
});
``
``
*/
unstubGlobals: true
```

</details>

