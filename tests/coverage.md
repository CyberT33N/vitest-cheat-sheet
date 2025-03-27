# Coverage

Vitest unterstützt das Generieren von Code-Coverage-Berichten, um die Testabdeckung des Codes zu messen.

Offizielle Dokumentation: [https://vitest.dev/guide/coverage.html#coverage-setup](https://vitest.dev/guide/coverage.html#coverage-setup)

## Konfiguration

Die Coverage-Konfiguration kann in der Vitest-Konfigurationsdatei vorgenommen werden:

```typescript
export default defineConfig({
    test: {
        coverage: {
            provider: 'v8', // oder 'istanbul'
            include: ['src/'],
            exclude: ['dist/'],
            reporter: ['text', 'json', 'html']
        }
    }
})
```

## Ausführung via Kommandozeile

Coverage kann auch über die Kommandozeile aktiviert werden:

```shell
vitest --coverage
```

Das Flag kann auch mit einem Alias verwendet werden:

```shell
vitest run --coverage
``` 