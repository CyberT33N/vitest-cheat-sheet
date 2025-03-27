# vitest.integration.config.ts

Spezifische Konfigurationsdatei für Integrationstests, die auf der Basis-Konfiguration aufbaut.

## Konfiguration in package.json

In der `package.json` kann der folgende Eintrag hinzugefügt werden, um die Integrationstests auszuführen:

```json
"test:integration": "vitest run --testTimeout=300000 --coverage --disable-console-intercept --watch=false --config vitest.integration.config.ts"
```

## Konfigurationsdatei

```typescript
// ==== DEPENDENCIES ====
import {
     defineConfig, mergeConfig,
     type ViteUserConfig
} from 'vitest/config'

// 🔌 Import the base Vitest configuration
import baseConfig from './vitest.config'

// 📋 Define the integration test configuration
const cfg: ViteUserConfig = defineConfig({
     test: {
         /**   
          * Specifies the test files to include.
          * @type {Array<string>}
          */
         include: ['test/integration/**/*.test.ts'],

         /**    
          * Specifies the global setup file to use.
          * @type {string}
          */
         globalSetup: 'test/integration/pretestAll.ts',
     }
 })

/**
 * 🛠️ Merges the existing Vitest configuration with additional custom 
 * configurations defined below.
 */
// Zuerst die Konfigurationen mergen
const mergedCfg = mergeConfig(baseConfig, defineConfig(cfg))

// Dann explizit das coverage.include überschreiben
if (mergedCfg.test?.coverage) {
    mergedCfg.test.coverage.include = ['src/controllers/']
}

export default mergedCfg 