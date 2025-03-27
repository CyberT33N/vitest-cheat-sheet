# vitest.unit.config.ts

Spezifische Konfigurationsdatei für Unit-Tests, die auf der Basis-Konfiguration aufbaut.

## Konfiguration in package.json

In der `package.json` kann der folgende Eintrag hinzugefügt werden, um die Unit-Tests auszuführen:

```json
"test:unit": "vitest run --testTimeout=300000 --coverage --disable-console-intercept --watch=false --config vitest.unit.config.ts"
```

## Konfigurationsdatei

```typescript
// ==== DEPENDENCIES ====
import {
     defineConfig, mergeConfig,
     type ViteUserConfig
} from 'vitest/config'

// 🔌 Import the base Vitest configuration
import vitestConfig from './vitest.config'

// 📋 Define the unit test configuration
const cfg: ViteUserConfig = defineConfig({
     test: {
         /** 
          * Specifies the test files to include.
          * @type {Array<string>}
          */
         include: ['test/unit/**/*.test.ts'],

         /** 
          * Specifies the setup file to use.
          * @type {string}
          */
         setupFiles: 'test/unit/pretestEach.ts',

         /** 
          * Specifies the coverage configuration.
          * @type {Object}
          */
         coverage: {
             /** 
              * Specifies the coverage provider to use.
              * @type {string}
              */
             provider: 'v8',

             /**
              * Specifies the files or directories to exclude from coverage.
              * @type {Array<string>}
              */
             exclude: ['src/controllers/']
         }
     }
})

/**
 * 🛠️ Merges the existing Vitest configuration with additional custom 
 * configurations defined below.
 */
const mergedCfg = mergeConfig(vitestConfig, defineConfig(cfg))
export default mergedCfg 