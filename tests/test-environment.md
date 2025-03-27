# Test Environment

Vitest bietet die Möglichkeit, die Testumgebung über die `environment`-Option zu konfigurieren. Das Verhalten der Umgebung kann mit der `environmentOptions`-Option angepasst werden.

Offizielle Dokumentation: [https://vitest.dev/guide/environment.html#test-environment](https://vitest.dev/guide/environment.html#test-environment)

## Verfügbare Umgebungen

Standardmäßig können folgende Umgebungen verwendet werden:

- `node` ist die Standardumgebung
- `jsdom` emuliert eine Browser-Umgebung durch Bereitstellung der Browser-API, verwendet das jsdom-Paket
- `happy-dom` emuliert eine Browser-Umgebung durch Bereitstellung der Browser-API und gilt als schneller als jsdom, hat jedoch einige fehlende APIs, verwendet das happy-dom-Paket
- `edge-runtime` emuliert Vercels edge-runtime, verwendet das @edge-runtime/vm-Paket

## populateGlobal

Die `populateGlobal`-Funktion kann verwendet werden, um globale Variablen in der Testumgebung zu definieren.

**Hinweis**: Dies kann in einigen Fällen zu Problemen führen, wenn man glob verwendet. Der genaue Grund dafür ist nicht bekannt.

### Beispiel für setupFiles

```typescript
// ==== DEPENDENCIES ====
import { beforeAll } from 'vitest'
import { populateGlobal } from 'vitest/environments'

// ==== INTERNAL ====
declare global {
    var modelDetails: {
        test: number
    };
}

beforeAll(async () => {
    const modelDetails = {
       test: 1234
    } 

    populateGlobal(global, { modelDetails })
})
```

### Nutzung in Tests

```typescript
test('sollte auf globale Variable zugreifen', () => {
  expect(global.modelDetails.test).toBe(1234);
});
``` 