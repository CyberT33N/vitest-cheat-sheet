# vitest.config.ts

Die Hauptkonfigurationsdatei für Vitest, die die grundlegenden Einstellungen für die Testumgebung definiert.

```typescript
// ==== DEPENDENCIES ====
import tsconfigPaths from 'vite-tsconfig-paths'
import { defineConfig, type ViteUserConfig } from 'vitest/config'

// 🌐 Import dotenv to load environment variables from .env files
import dotenv from 'dotenv'

// 🔧 Load the default environment variables from the .env file
dotenv.config()

// 🔄 Load the test environment variables from .env.test and override defaults
dotenv.config({ path: '.env.test', override: true })

const cfg: ViteUserConfig = defineConfig({
     /** 
      * List of plugins to be used in the configuration.
      */
     plugins: [tsconfigPaths()], // 🔌 Add tsconfig paths plugin
 
     /** 
      * Configuration options for tests.
      */
     test: {
         /** 
          * Indicates whether to watch files for changes.
          * @type {boolean}
          */
         watch: false, // 🚫 Disable watch mode
 
         /** 
          * Path to the setup file that runs before each test. 
          * @type {string}
          */
         setupFiles: 'test/unit/pretestEach.ts',
 
         /** 
          * Path to the global setup file for integration tests.
          * @type {string}
          */
         globalSetup: 'test/integration/pretestAll.ts',
 
         /** 
          * The environment in which the tests will run.
          * @type {string}
          */
         environment: 'node', // 🌐 Test environment set to Node.js
 
         typecheck: {
             enabled: true,
             // 🔍 Specify files to include for type checking
             include: ['**/*.{test,spec}.?(c|m)[jt]s?(x)']
         },
 
         /** 
          * Configuration for coverage reporting.
          */
         coverage: {
             /** 
              * Specifies the coverage provider to use.
              * @type {string}
              */
             provider: 'v8',

             /** 
              * Specifies the directories to include for coverage.
              * @type {Array<string>}
              */
             include: ['src/'],
 
             /** 
              * Specifies the files or directories to exclude from coverage.
              * @type {Array<string>}
              */
             exclude: ['dist/'],
 
             /** 
              * Specifies the coverage reporters to use.
              * @type {Array<string>}
              */
             reporter: ['text', 'json', 'html']
         }
     }
})

/**
 * Represents the configuration for the Vitest test runner.
 */
export default cfg 