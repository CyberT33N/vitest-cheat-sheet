# Testing Types

Vitest ermöglicht das Schreiben von Tests für Typen mit den Syntaxen `expectTypeOf` oder `assertType`. Standardmäßig werden alle Tests in `*.test-d.ts`-Dateien als Typ-Tests betrachtet, aber dies kann mit der Konfigurationsoption `typecheck.include` geändert werden.

Offizielle Dokumentation: [https://vitest.dev/guide/testing-types](https://vitest.dev/guide/testing-types)

## Konfiguration für reguläre .test.ts-Dateien

Hier ist ein Beispiel, um alle `.test.ts`-Dateien statt nur `*.test-d.ts` für Typen-Tests zu verwenden. Es wird jedoch empfohlen, eigene Testdateien für Typprüfungen zu erstellen, anstatt alles in einer einzigen Datei zu bündeln.

```typescript
import dotenv from 'dotenv'
// Load .env 
dotenv.config()
// Load .env.test and override .env
dotenv.config({ path: '.env.test', override: true })

// ==== DEPENDENCIES ====
import tsconfigPaths from 'vite-tsconfig-paths'

// ==== VITEST ====
import { defineConfig } from 'vitest/config'

/**
 * Represents the configuration for the Vitest test runner.
 */
export default defineConfig({
    plugins: [tsconfigPaths()],
    test: {
        watch: false,
        setupFiles: 'test/unit/pretestEach.ts',
        globalSetup: 'test/integration/pretestAll.ts',
        environment: 'node',
        typecheck: {
            include: ['**/*.{test,spec}.?(c|m)[jt]s?(x)'] // Hier den typecheck.include Wert einfügen
        },
        coverage: {
            /**
             * Specifies the directories to include for coverage.
             */
            include: ['src/'],
            /**
             * Specifies the files or directories to exclude from coverage.
             */
            //exclude: ['src/legacy/', 'utils/helpers.ts'],
            /**
             * Specifies the coverage reporters to use.
             */
            reporter: ['text', 'json', 'html']
        }
    }
})
```

## Hintergrundfunktionsweise

Intern ruft Vitest `tsc` oder `vue-tsc` auf, abhängig von der Konfiguration, und parst die Ergebnisse. Vitest gibt auch Typfehler im Quellcode aus, wenn welche gefunden werden. Dies kann mit der Konfigurationsoption `typecheck.ignoreSourceErrors` deaktiviert werden.

Beachten Sie, dass Vitest diese Dateien nicht ausführt, sondern nur statisch vom Compiler analysieren lässt. Das bedeutet, dass wenn dynamische Namen, `test.each` oder `test.for` verwendet werden, der Testname nicht ausgewertet wird - er wird unverändert angezeigt.

## Aktivieren über Kommandozeile

Um Typprüfungen zu aktivieren, fügen Sie einfach das Flag `--typecheck` zu Ihrem Vitest-Befehl in package.json hinzu:

```json
{
  "scripts": {
    "test": "vitest --typecheck"
  }
}
``` 