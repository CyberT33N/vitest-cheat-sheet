# VI Utility

Vitest stellt Hilfsfunktionen über den `vi`-Helper bereit. Sie können darauf global zugreifen (wenn die `globals`-Konfiguration aktiviert ist) oder ihn direkt von Vitest importieren:

```typescript
import { vi } from 'vitest'
```

Offizielle Dokumentation: [https://vitest.dev/api/vi.html](https://vitest.dev/api/vi.html)

## vi.stubEnv

Ändert den Wert einer Umgebungsvariable in `process.env` und `import.meta.env`. Sie können den ursprünglichen Wert durch Aufruf von `vi.unstubAllEnvs` wiederherstellen.

```typescript
import { vi } from 'vitest'

// `process.env.NODE_ENV` und `import.meta.env.NODE_ENV`
// sind "development" vor dem Aufruf von "vi.stubEnv"

vi.stubEnv('NODE_ENV', 'production')

process.env.NODE_ENV === 'production'
import.meta.env.NODE_ENV === 'production'

vi.stubEnv('NODE_ENV', undefined)

process.env.NODE_ENV === undefined
import.meta.env.NODE_ENV === undefined

// ändert keine anderen Umgebungsvariablen
import.meta.env.MODE === 'development'
```

## vi.unstubAllEnvs

Stellt alle `import.meta.env` und `process.env` Werte wieder her, die mit `vi.stubEnv` geändert wurden. Wenn es zum ersten Mal aufgerufen wird, merkt sich Vitest den Originalwert und speichert ihn, bis `unstubAllEnvs` erneut aufgerufen wird.

```typescript
import { vi } from 'vitest'

// `process.env.NODE_ENV` und `import.meta.env.NODE_ENV`
// sind "development" vor dem Aufruf von "vi.stubEnv"

vi.stubEnv('NODE_ENV', 'production')

process.env.NODE_ENV === 'production'
import.meta.env.NODE_ENV === 'production'

vi.unstubAllEnvs()

// Werte werden auf die ursprünglichen zurückgesetzt
process.env.NODE_ENV === 'development'
import.meta.env.NODE_ENV === 'development'
```

Offizielle Dokumentation: [https://vitest.dev/api/vi.html#vi-unstuballenvs](https://vitest.dev/api/vi.html#vi-unstuballenvs) 