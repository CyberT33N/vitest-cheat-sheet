# Nested Tests

In Vitest können Tests verschachtelt werden, indem vordefinierte Testdateien vor den eigentlichen Tests importiert werden.

## Beispiel

Wenn eine `pretest.ts`-Datei vor `main.test.ts` ausgeführt werden soll, kann das wie folgt umgesetzt werden:

### pretest.ts

```typescript
// ==== VITEST ====
import { beforeAll } from 'vitest'

beforeAll(() => {
    // ..
})
```

### main.test.ts

#### Alternative #1

```typescript
// ==== VITEST ====
import { beforeAll } from 'vitest'

describe('[INTEGRATION] - src/errors/BaseError', () => {
    beforeAll(async() => {
        await import('./pretest')
    })

    it('should return 500 with BaseError details - error passed', async() => {
        // ..
    })
})
```

#### Alternative #2

```typescript
await import('./pretest')

describe('[INTEGRATION] - src/errors/BaseError', () => {
    it('should return 500 with BaseError details - error passed', async() => {
        // ..
    })
})
``` 