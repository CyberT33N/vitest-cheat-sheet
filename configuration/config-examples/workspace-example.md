# Workspace Method (--project) - recommended
- Diese Methode benutzt den Vitest Workspace. Dadurch definieren wir einmal einen Unit-Test und ein Integrations-Test-Projekt. In der Package JSON werden diese dann über das Comment-Line flag project gesetzt. Obwohl dieses Prinzip für Monorepos benutzt wird, benutzen selbst die Ersteller von V-Test und andere große Projekte wie Vue.js dieses Prinzip auch, um in einem normalen Repo zu differenzieren zwischen verschiedenen Testarten wie End-to-End oder Unit-Test-Configs. Daher ist dies ein valider Ansatz.

- https://vitest.dev/guide/workspace.html


```text
Some of the configuration options are not allowed in a project config. Most notably:

coverage: coverage is done for the whole workspace
reporters: only root-level reporters can be supported
resolveSnapshotPath: only root-level resolver is respected
all other options that don't affect test runners
```

## Examples other projects
- https://github.com/vuejs/core/tree/main
- https://github.com/vitest-dev/vitest/tree/main/test/workspaces



<details><summary>Click to expand..</summary>


# package.json

```json
"scripts": {
  "test": "vitest --typecheck --coverage",
  "test:watch": "vitest --typecheck --watch",
  "test:integration": "vitest run --project integration",
  "test:unit": "vitest run --project unit",
  "test:production": "vitest run --project production"
}
```


# test/unit/test-setup.ts

<details><summary>Click to expand..</summary>



```typescript
/**
 * 📌 test/unit/test-setup.ts
 * 
 * Setup-Datei für Unit-Tests.
 * Diese Datei wird in der setupFiles-Konfiguration der vitest.unit.config.ts geladen.
 * 
 * WICHTIG: Hier können vi.* Funktionen verwendet werden, da setupFiles
 * im Kontext der Testsuite ausgeführt wird.
 */

import { vi } from 'vitest'

/**
 * 🧪 Unit-Test-Setup-Logik
 * Diese Funktion bereitet die Umgebung für Unit-Tests vor
 */
function setupUnitTestEnvironment(): void {
    console.info('🧪 Initialisiere Unit-Test-Umgebung...')
    
    // Hier können Vitest-spezifische Mocks und Setups erfolgen
    vi.stubGlobal('UNIT_TEST_MODE', true)
    
    // Eigene Mocks oder Erweitern der vorhandenen Electron-Mocks spezifisch für Unit-Tests
    // Express-Komponenten müssen in Unit-Tests grundsätzlich gemockt werden
    vi.mock('express', () => ({
        // eslint-disable-next-line @typescript-eslint/naming-convention
        Router: vi.fn(() => ({
            get: vi.fn(),
            post: vi.fn(),
            put: vi.fn(),
            delete: vi.fn()
        }))
    }))
    
    console.info('✅ Unit-Test-Umgebung erfolgreich initialisiert')
}

// Automatische Ausführung beim Import
setupUnitTestEnvironment() 
```

</details>




<br><br>


# test/unit/pretestEach.ts

<details><summary>Click to expand..</summary>



```typescript

// ==== DEPENDENCIES ====
import { beforeEach } from 'vitest'
import '../utils/setup-electron-mock' // Initialisiere Electron-Mocks

/**
 * 🔧 Sets up the test environment before each test.
 * 
 * @returns {void} A promise that resolves when the test environment is set up.
 */
beforeEach((): void => {
    // vi.unstubAllEnvs()
})
```

</details>




<br><br>












# test/integration/test-setup.ts

<details><summary>Click to expand..</summary>



```typescript
/**
 * 📌 test/integration/test-setup.ts
 * 
 * Setup-Datei für Integrationstests.
 * Diese Datei wird in der setupFiles-Konfiguration der vitest.integration.config.ts geladen.
 * 
 * WICHTIG: Hier können vi.* Funktionen verwendet werden, da setupFiles
 * im Kontext der Testsuite ausgeführt wird.
 */

import { vi, afterAll } from 'vitest'
import { ApiController } from '@/main/controllers/ApiController.ts'

// Globale Variable für den API-Controller (wird für cleanup benötigt)
let apiController: ApiController | null = null
const serverPort = process.env.SERVER_PORT ?? '9090'

/**
 * 🧪 Integrations-Test-Setup-Logik
 * Diese Funktion bereitet die Umgebung für Integrationstests vor
 */
function setupIntegrationTestEnvironment(): void {
    console.info('📋 [4. SETUP-FILES-INTEGRATION] Initialisiere Integration-Test-Umgebung...')
    
    // Hier können Vitest-spezifische Mocks und setups erfolgen
    vi.stubGlobal('INTEGRATION_TEST_MODE', true)
    
    // Eigene Mocks oder Erweitern der vorhandenen Electron-Mocks spezifisch für Integration-Tests
    
    console.info('✅ [4. SETUP-FILES-INTEGRATION] Integration-Test-Umgebung erfolgreich initialisiert')
    
    // Starte den API-Server, wenn das Flag gesetzt ist
    // Hinweis: Dies geschieht hier, NACHDEM die Electron-Mocks initialisiert wurden
    if (process.env.INTEGRATION_TEST_START_SERVER === 'true') {
        try {
            console.info(`📋 [4.1 SERVER-START] Starte Integration-Test-Server auf Port ${serverPort}...`)
            
            // Initialisiere den API-Controller mit den aktiven Electron-Mocks
            apiController = new ApiController()
            apiController.startServer()
            
            console.info(`✅ [4.1 SERVER-START] Integration-Test-Server erfolgreich gestartet`)
        } catch (error) {
            console.error('❌ [4.1 SERVER-START] Fehler beim Starten des Integration-Test-Servers:', error)
            throw error
        }
    }
}

/**
 * 🧹 Cleanup-Logik für die Integrationstests
 * Diese Funktion wird nach allen Tests ausgeführt
 */
function cleanupIntegrationTestEnvironment(): void {
    console.info('🧹 [CLEANUP-INTEGRATION] Räume Integration-Test-Umgebung auf...')
    
    // Fahre den API-Server herunter, wenn er läuft
    if (apiController) {
        console.info('🔄 [SERVER-STOP] Stoppe Integration-Test-Server...')
        try {
            apiController.app.listen().close()
            apiController = null
            console.info('✅ [SERVER-STOP] Integration-Test-Server erfolgreich gestoppt')
        } catch (error) {
            console.error('❌ [SERVER-STOP] Fehler beim Stoppen des Integration-Test-Servers:', error)
        }
    }
    
    console.info('✅ [CLEANUP-INTEGRATION] Integration-Test-Umgebung erfolgreich aufgeräumt')
}

// Automatische Ausführung beim Import
setupIntegrationTestEnvironment()

// Registriere die Cleanup-Funktion, die nach allen Tests ausgeführt wird
afterAll(() => {
    console.info('🧹 [AFTER-ALL] Starte afterAll-Cleanup für Integration-Tests...')
    cleanupIntegrationTestEnvironment()
    console.info('✅ [AFTER-ALL] afterAll-Cleanup für Integration-Tests abgeschlossen')
}) 
```

</details>




<br><br>




# test/integration/pretestAll.ts

<details><summary>Click to expand..</summary>



```typescript

/**
 * 📌 pretestAll.ts
 * 
 * Integrations-Test Setup-Datei.
 * Diese Datei wird als globalSetup in vitest.integration.config.ts eingetragen.
 * 
 * WICHTIG: Hier dürfen KEINE vi.* Funktionen verwendet werden, da globalSetup
 * in einem separaten Kontext ausgeführt wird.
 */

// ⚠️ WICHTIG: KEINE direkten Imports von Electron-abhängigen Modulen hier,
// da die Electron-Mocks erst in setupFiles (nach globalSetup) initialisiert werden!

// Umgebungsvariablen werden automatisch von vitest.config.ts geladen

/**
 * ⚠️ Hinweis: Die Electron-Mocks werden durch setupFiles geladen
 * Da in globalSetup keine vi.mock() verfügbar ist, werden die Mocks
 * über die setupFiles-Konfiguration in vitest.integration.config.ts eingebunden.
 * 
 * Der ApiController wird erst in den Tests selbst initialisiert, da er
 * Electron-Abhängigkeiten hat. Dort sind die Mocks dann bereits aktiv.
 */

/**
 * 🔧 Sets up the integration test environment
 * @returns {void} A promise that resolves when the test environment is set up.
 */
export function setup(): void {
    console.info('📋 [2. PRETEST-INTEGRATION] Starte Integration-Test-Setup...')
    
    // Speichere Marker für die Test-Setup-Datei, dass sie den Server starten soll
    process.env.INTEGRATION_TEST_START_SERVER = 'true'
    process.env.SERVER_PORT = process.env.SERVER_PORT ?? '9090'
    
    console.info(
        '✅ [2. PRETEST-INTEGRATION] Integration-Test-Setup abgeschlossen - ' +
        `Server wird im Test-Kontext auf Port ${process.env.SERVER_PORT} gestartet`
    )
}

/**
 * 🔒 Cleans up the integration test environment
 * @returns {void} A promise that resolves when the test environment is cleaned up.
 */
export function teardown(): void {
    console.info('🧹 [TEARDOWN - INTEGRATION] Starte Integration-Test-Cleanup...')
    
    // API-Server wird in test-setup.ts heruntergefahren, da er auch dort gestartet wurde
    // Dies stellt sicher, dass sowohl Server-Start als auch -Stop im selben Kontext (mit Mocks) stattfinden
    
    console.info('✅ [TEARDOWN - INTEGRATION] Integration-Test-Cleanup abgeschlossen')
}
```

</details>




<br><br>










# test/pretest-base.ts

<details><summary>Click to expand..</summary>



```typescript
/**
 * 📌 pretest-base.ts
 * 
 * Basis-Setup-Datei für alle Tests (Unit und Integration).
 * Diese Datei wird als globalSetup in vitest.config.ts eingetragen und
 * stellt grundlegende Test-Infrastruktur bereit.
 * 
 * Hinweis: In dieser Datei können KEINE Vitest-spezifischen Funktionen wie
 * vi, expect, etc. verwendet werden, da sie in einem separaten Kontext ausgeführt wird.
 */

// Import des zentralen Bootstrap
import { bootstrapTestEnvironment, cleanupTestEnvironment } from './bootstrap.ts'

/**
 * 🔄 Setup-Funktion für Vitest
 * Diese Funktion wird vor allen Tests ausgeführt
 */
export function setup(): void {
    console.info('📋 [1. PRETEST-BASE] Starte Basis-Test-Setup...')
    
    // Zentralen Bootstrap ausführen
    bootstrapTestEnvironment()
    
    // Zusätzliches spezifisches Setup für die Basis-Konfiguration
    console.info('✅ [1. PRETEST-BASE] Basis-Test-Setup abgeschlossen')
}

/**
 * 🧹 Teardown-Funktion für Vitest
 * Diese Funktion wird nach allen Tests ausgeführt
 */
export function teardown(): void {
    console.info('🧹 [TEARDOWN - PRETEST-BASE] Starte Basis-Test-Teardown...')
    
    // Zentralen Cleanup ausführen
    cleanupTestEnvironment()
    
    // Zusätzliches spezifisches Cleanup für die Basis-Konfiguration
    console.info('✅ [TEARDOWN - PRETEST-BASE] Basis-Test-Teardown abgeschlossen')
} 
```

</details>




<br><br>












# test/bootstrap.ts

<details><summary>Click to expand..</summary>



```typescript
/**
 * 🧰 test/bootstrap.ts
 * 
 * Zentrale Bootstrap-Datei für alle Testarten.
 * Diese Datei wird von allen globalSetup-Dateien importiert und stellt sicher,
 * dass grundlegende Einstellungen vor testspezifischem Setup initialisiert werden.
 * 
 * WICHTIG: Diese Datei darf KEINE vi.* Funktionen verwenden, da sie in einem
 * globalSetup-Kontext ausgeführt wird, in dem Vitest-Funktionalitäten nicht verfügbar sind.
 */

/**
 * 🧪 Initialisiert die grundlegende Testumgebung
 * @returns {void}
 */
export function bootstrapTestEnvironment(): void {
    console.info('📋 [1.1 BOOTSTRAP] Initialisiere grundlegende Testumgebung...')
    
    // Setze Test-Modus-Flag explizit für alle Tests
    process.env.__TEST_MODE__ = 'true'
    
    // Erzwinge bestimmte Umgebungsvariablen für Tests
    process.env.NODE_ENV = 'test'
    
    // Verhindere Benutzerinteraktion in Tests
    process.env.CI = 'true'
    
    // Setze Testspezifische Flags (können in Tests abgefragt werden)
    process.env.ELECTRON_MOCKS_ENABLED = 'true'
    
    console.info('✅ [1.1 BOOTSTRAP] Grundlegende Testumgebung erfolgreich initialisiert')
}

/**
 * 🧹 Räumt die Testumgebung auf
 * @returns {void}
 */
export function cleanupTestEnvironment(): void {
    console.info('🧹 [FINAL CLEANUP] Bootstrap-Test-Environment-Cleanup wird ausgeführt...')
    
    // Hier können allgemeine Cleanup-Operationen erfolgen
    
    console.info('✅ [FINAL CLEANUP] Bootstrap-Test-Environment-Cleanup abgeschlossen')
} 
```

</details>




<br><br>















# vitest.integration.config.ts

<details><summary>Click to expand..</summary>


```typescript

// ==== DEPENDENCIES ====
import {
    defineConfig, mergeConfig,
    type ViteUserConfig
} from 'vitest/config'

// 🔌 Import the base Vitest configuration
import baseConfig from './vitest.config.ts'

// 📋 Define the integration test configuration
const cfg: ViteUserConfig = defineConfig({
    test: {
        /**
          * Name of the test configuration for workspace selection.
          * @type {string}
          */
        name: 'integration',
        /**   
           * Specifies the test files to include.
           * @type {Array<string>}
           */
        include: ['test/integration/**/*rest.test.ts'],
        /**    
           * Specifies the global setup file to use for integration tests.
           * Die Basis-Setup-Datei (pretest-base.ts) wird bereits in der Basis-Konfiguration geladen.
           * Diese Datei setzt nur die Integration-spezifischen Parameter.
           * @type {string}
           */
        globalSetup: ['test/integration/pretestAll.ts'],
        /**
           * Specifies the setup files to run before each test file.
           * Diese Dateien werden NACH globalSetup ausgeführt und können 
           * Vitest-Funktionalitäten wie vi.mock() verwenden.
           * @type {Array<string>}
           */
        setupFiles: [
            // Integration-spezifisches Setup (falls notwendig)
            'test/integration/test-setup.ts'
        ],
        /**
           * Type checking configuration for integration tests.
           * @type {Object}
           */
        typecheck: {
            /** 
              * Specifies the files to include for type checking.
              * @type {Array<string>}
              */
            include: ['test/integration/**/*rest.test.ts']
        }
    }
})

/**
 * 🛠️ Merges the existing Vitest configuration with additional custom 
 * configurations defined below.
 */
const mergedCfg: ViteUserConfig = mergeConfig(baseConfig, defineConfig(cfg))
console.info('mergedCfg - integration config', mergedCfg)
export default mergedCfg
```

</details>




<br><br>




# vitest.unit.config.ts

<details><summary>Click to expand..</summary>


```typescript

// ==== DEPENDENCIES ====
import {
    defineConfig, mergeConfig,
    type ViteUserConfig
} from 'vitest/config'

// 🔌 Import the base Vitest configuration
import baseConfig from './vitest.config.ts'

// 📋 Define the unit test configuration
const cfg: ViteUserConfig = defineConfig({
    test: {
        /**
          * Name of the test configuration for workspace selection.
          * @type {string}
          */
        name: 'unit',
        /** 
          * Specifies the test files to include.
          * @type {Array<string>}
          */
        include: ['test/unit/**/*.test.ts'],
        /** 
          * Specifies the setup files to use for unit tests.
          * Der Electron-Mock wird bereits in der Basiskonfiguration geladen.
          * @type {Array<string>}
          */
        setupFiles: ['test/unit/test-setup.ts'],
        /** 
          * Type checking configuration for unit tests.
          * @type {Object}
          */
        typecheck: {
            /** 
              * Specifies the files to include for type checking.
              * @type {Array<string>}
              */
            include: ['test/unit/**/*.test.ts']
        },
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
            exclude: ['src/main/controllers/']
        }
    }
})

/**
 * 🛠️ Merges the existing Vitest configuration with additional custom 
 * configurations defined below.
 */
const mergedCfg = mergeConfig(baseConfig, defineConfig(cfg))
console.info('mergedCfg - unit config', mergedCfg)
export default mergedCfg
```

</details>



<br><br>




# vitest.workspace.ts

<details><summary>Click to expand..</summary>


```typescript
import { defineWorkspace } from 'vitest/config'

/**
 * Workspace configuration for Vitest to manage different test projects.
 * This allows running tests with specific configurations using the --project flag.
 */
export default defineWorkspace([
    './vitest.unit.config.ts',
    './vitest.integration.config.ts'
]) 
```

</details>






<br><br>




# vitest.config.ts

<details><summary>Click to expand..</summary>


```typescript
// ==== DEPENDENCIES ====
import dotenv from 'dotenv'
import tsconfigPaths from 'vite-tsconfig-paths'
import { defineConfig, type ViteUserConfig } from 'vitest/config'

// 🌐 Import dotenv to load environment variables from .env files

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
        // ✅ Sauberes Mocking, um Nebeneffekte zu vermeiden
        // In vitest.config.ts
        clearMocks: true,     // Aufrufe zurücksetzen
        restoreMocks: false,  // Mock-Implementierungen behalten
        mockReset: false,     // Mock-Definitionen nicht zurücksetzen

        // ✅ Verhindert, dass Tests ungenutzte Umgebungsvariablen beeinflussen
        unstubEnvs: true,
        unstubGlobals: true,

        /** 
          * Indicates whether to watch files for changes.
          * @type {boolean}
          */
        watch: false,
 
        /** 
          * Path to the setup file that runs before each test.
          * Initialisiert die Electron-Mocks und andere gemeinsame Testfunktionalitäten.
          * @type {string}
          */
        setupFiles: ['test/utils/setup-electron-mock.ts'],
 
        /** 
          * Path to the global setup file that runs before all tests.
          * Handles basic test infrastructure like environment variables.
          * @type {string}
          */
        globalSetup: ['test/pretest-base.ts'],
 
        /** 
          * The environment in which the tests will run.
          * @type {string}
          */
        environment: 'node', // 🌐 Test environment set to Node.js

        /**
          * Disables the console intercept.
          * @type {boolean}
          */
        disableConsoleIntercept: true,
 
        typecheck: {
            enabled: true
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
            exclude: ['dist/', 'out/', 'log/', '.cursor/'],
 
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

```

</details>


</details>
