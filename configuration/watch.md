# Watch Mode

Der Watch-Modus in Vitest kann über die Konfigurationsdatei gesteuert werden:

```typescript
export default defineConfig({
    test: {
        watch: false, // Deaktiviert den Watch-Modus
    }
})
```

Alternativ kann der Watch-Modus auch über die Kommandozeile gesteuert werden:

```shell
--watch=false
```

oder

```shell
--run
``` 