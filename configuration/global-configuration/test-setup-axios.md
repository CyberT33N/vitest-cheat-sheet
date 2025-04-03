# Globale Konfiguration mit Axios

Dieses Beispiel zeigt, wie man eine globale Axios-Instanz für Tests konfiguriert und in allen Tests verfügbar macht.

## TypeScript Deklaration (global.d.ts)

Zunächst muss die globale Variable in einer TypeScript-Deklarationsdatei definiert werden:

```typescript
import { AxiosInstance } from 'axios'

export declare global {
  // eslint-disable-next-line no-var
  var axiosP1: AxiosInstance
}
```

## Test-Setup (test-setup.ts)

Im Test-Setup wird die Axios-Instanz mit `beforeAll` global initialisiert:

```typescript
// Setze die axiosP1-Instanz mit beforeAll
beforeAll(() => {
    console.info('📋 [4.0 BEFORE-ALL] Erstelle globale axiosP1-Instanz...')
    
    // API-Key aus der Umgebungsvariable lesen, die in pretestAll.ts gesetzt wurde
    const API_KEY = process.env.TEST_API_KEY ?? 'test-api-key-0123456789abcdef'
    
    // Erstelle eine vorkonfigurierte Axios-Instanz
    globalThis.axiosP1 = axios.create({
        baseURL: `http://localhost:${serverPort}`,
        headers: {
            'API-KEY': API_KEY
        }
    })
    
    console.info('✅ [4.0 BEFORE-ALL] Globale axiosP1-Instanz erfolgreich erstellt')
})
```

## Vitest Konfiguration (vitest.integration.config.ts)

Schließlich muss die Setup-Datei in der Vitest-Konfiguration angegeben werden:

```typescript
// In vitest.integration.config.ts
export default defineConfig({
    // andere Konfigurationen...
    setupFiles: [
        // Integration-spezifisches Setup (falls notwendig)
        'test/integration/test-setup.ts'
    ],
    // weitere Konfigurationen...
})
```

## Verwendung in Tests

Nach dieser Konfiguration kann die globale Axios-Instanz in jedem Test verwendet werden:

```typescript
test('sollte einen API-Endpunkt aufrufen', async () => {
    const response = await global.axiosP1.get('/api/data');
    expect(response.status).toBe(200);
});
```

## Vorteile

- Zentrale Konfiguration der Axios-Instanz
- Konsistente Basis-URL und Headers in allen Tests
- Typsicherheit durch TypeScript-Deklaration
- Einfache Wartung durch zentralisiertes Setup 