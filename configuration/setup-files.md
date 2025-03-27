# setupFiles

Die `setupFiles`-Option in der Vitest-Konfiguration definiert den Pfad zu Setup-Dateien, die vor jedem Testfile ausgeführt werden.

Offizielle Dokumentation: [https://vitest.dev/config/#setupfiles](https://vitest.dev/config/#setupfiles)

## Beispiel-Konfiguration

```typescript
export default defineConfig({
    // esbuild: { target: 'ES2022' },
    plugins: [tsconfigPaths(), react()],
    test: {
        environment: 'node',
        setupFiles: 'test/setup-tests-beforeEach.ts',
        globalSetup: 'test/setup-tests.ts',
        coverage: {
            // Include only specific directories for coverage
            include: ['app/api/', 'src/', 'utils/'],
            // Optional: Exclude certain files or directories
            //exclude: ['src/legacy/', 'utils/helpers.ts'],
            // Optional: Specify coverage reporters (e.g., text, json, html)
            reporter: ['text', 'json', 'html']
        }
    }
})
```

## Beispiel-Setup-Datei

```typescript
// ==== VITEST ====
import { beforeEach} from 'vitest'

const NLE = process.env.npm_lifecycle_event

beforeEach(() => {
    process.env.npm_lifecycle_event = NLE
})
```

## Definieren von Variablen in setupFiles und Verwendung in Tests

Man kann globale Variablen in Setup-Dateien definieren und in allen Tests verwenden:

```typescript
// setup-teardown-hook.js
import { afterAll, beforeAll } from 'vitest';


// ==== INTERNAL ====
declare global {
    var lol: {
        test: number
    };
}


beforeAll(() => {
  global.lol = 123;
});
afterAll(() => {
  delete global.lol
});
``` 