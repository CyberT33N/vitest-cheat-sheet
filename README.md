# Vitest cheat sheet





<br><br>
<br><br>


# Install

## Next.js
```shell
npm install -D vitest @vitejs/plugin-react jsdom @testing-library/react vite-tsconfig-paths @vitest/coverage-v8
```

- vitest.config.ts:
```typescript
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import tsconfigPaths from 'vite-tsconfig-paths'
 
export default defineConfig({
    plugins: [tsconfigPaths(), react()],
    test: {
        environment: 'jsdom'
    }
})
```














<br><br>
<br><br>
___________________________________
___________________________________
<br><br>
<br><br>


# CLI

<details><summary>Click to expand..</summary>

```
Usage:
  $ vitest [...filters]

Commands:
  run [...filters]        
  related [...filters]    
  watch [...filters]      
  dev [...filters]        
  bench [...filters]      
  typecheck [...filters]  
  [...filters]            

For more info, run any command with the `--help` flag:
  $ vitest run --help
  $ vitest related --help
  $ vitest watch --help
  $ vitest dev --help
  $ vitest bench --help
  $ vitest typecheck --help
  $ vitest --help
  $ vitest --help --expand-help

Options:
  -v, --version                                  Display version number 
  -r, --root <path>                              Root path 
  -c, --config <path>                            Path to config file 
  -u, --update                                   Update snapshot 
  -w, --watch                                    Enable watch mode 
  -t, --testNamePattern <pattern>                Run tests with full names matching the specified regexp pattern 
  --dir <path>                                   Base directory to scan for the test files 
  --ui                                           Enable UI 
  --open                                         Open UI automatically (default: !process.env.CI) 
  --api [port]                                   Specify server port. Note if the port is already being used, Vite will automatically try the next available port so this may not be the actual port the server ends up listening on. If true will be set to 51204. Use '--help --api' for more info. 
  --silent                                       Silent console output from tests 
  --hideSkippedTests                             Hide logs for skipped tests 
  --reporter <name>                              Specify reporters 
  --outputFile <filename/-s>                     Write test results to a file when supporter reporter is also specified, use cac's dot notation for individual outputs of multiple reporters (example: --outputFile.tap=./tap.txt) 
  --coverage                                     Enable coverage report. Use '--help --coverage' for more info. 
  --mode <name>                                  Override Vite mode (default: test or benchmark) 
  --workspace <path>                             Path to a workspace configuration file 
  --isolate                                      Run every test file in isolation. To disable isolation, use --no-isolate (default: true) 
  --globals                                      Inject apis globally 
  --dom                                          Mock browser API with happy-dom 
  --browser <name>                               Run tests in the browser. Equivalent to --browser.enabled (default: false). Use '--help --browser' for more info. 
  --pool <pool>                                  Specify pool, if not running in the browser (default: threads) 
  --poolOptions <options>                        Specify pool options. Use '--help --poolOptions' for more info. 
  --fileParallelism                              Should all test files run in parallel. Use --no-file-parallelism to disable (default: true) 
  --maxWorkers <workers>                         Maximum number of workers to run tests in 
  --minWorkers <workers>                         Minimum number of workers to run tests in 
  --environment <name>                           Specify runner environment, if not running in the browser (default: node) 
  --passWithNoTests                              Pass when no tests are found 
  --logHeapUsage                                 Show the size of heap for each test when running in node 
  --allowOnly                                    Allow tests and suites that are marked as only (default: !process.env.CI) 
  --dangerouslyIgnoreUnhandledErrors             Ignore any unhandled errors that occur 
  --shard <shards>                               Test suite shard to execute in a format of <index>/<count> 
  --changed [since]                              Run tests that are affected by the changed files (default: false) 
  --sequence <options>                           Options for how tests should be sorted. Use '--help --sequence' for more info. 
  --inspect [[host:]port]                        Enable Node.js inspector (default: 127.0.0.1:9229) 
  --inspectBrk [[host:]port]                     Enable Node.js inspector and break before the test starts 
  --testTimeout <timeout>                        Default timeout of a test in milliseconds (default: 5000) 
  --hookTimeout <timeout>                        Default hook timeout in milliseconds (default: 10000) 
  --bail <number>                                Stop test execution when given number of tests have failed (default: 0) 
  --retry <times>                                Retry the test specific number of times if it fails (default: 0) 
  --diff <path>                                  Path to a diff config that will be used to generate diff interface 
  --exclude <glob>                               Additional file globs to be excluded from test 
  --expandSnapshotDiff                           Show full diff when snapshot fails 
  --disableConsoleIntercept                      Disable automatic interception of console logging (default: false) 
  --typecheck                                    Enable typechecking alongside tests (default: false). Use '--help --typecheck' for more info. 
  --project <name>                               The name of the project to run if you are using Vitest workspace feature. This can be repeated for multiple projects: --project=1 --project=2. You can also filter projects using wildcards like --project=packages* 
  --slowTestThreshold <threshold>                Threshold in milliseconds for a test to be considered slow (default: 300) 
  --teardownTimeout <timeout>                    Default timeout of a teardown function in milliseconds (default: 10000) 
  --cache                                        Enable cache. Use '--help --cache' for more info. 
  --maxConcurrency <number>                      Maximum number of concurrent tests in a suite (default: 5) 
  --run                                          Disable watch mode 
  --segfaultRetry <times>                        Retry the test suite if it crashes due to a segfault (default: true) 
  --no-color                                     Removes colors from the console output (default: true)
  --clearScreen                                  Clear terminal screen when re-running tests during watch mode (default: true) 
  --standalone                                   Start Vitest without running tests. File filters will be ignored, tests will be running only on change (default: false) 
  -h, --help                                     Display this message 
```


<br><br>
<br><br>

## Enable console.log
- `--disable-console-intercept`

<br><br>

## Disable watch
- `--watch=false`

<br><br>

## timeout
- `--testTimeout=5000`

</details>






















<br><br>
<br><br>
___________________________________
___________________________________
<br><br>
<br><br>

# vitest.config.ts
- https://vitest.dev/config/

<details><summary>Click to expand..</summary>
 
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
```


</details>




















<br><br>
<br><br>
<br><br>
<br><br>

## vitest.unit.config.ts
- package.json add to scripts `"test:unit": "vitest run --testTimeout=300000 --coverage --disable-console-intercept --watch=false --config vitest.unit.config.ts"`

<details><summary>Click to expand..</summary>

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
```

</details>












<br><br>
<br><br>
<br><br>
<br><br>






## vitest.integration.config.ts
- package.json add to scripts `"test:integration": "vitest run --testTimeout=300000 --coverage --disable-console-intercept --watch=false --config vitest.integration.config.ts"`

<details><summary>Click to expand..</summary>

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
```



</details>




















<br><br>
<br><br>
<br><br>
<br><br>

## setupFiles
- https://vitest.dev/config/#setupfiles
- Path to setup files. They will be run before each test file.

<details><summary>Click to expand..</summary>

vitest.config.ts
```
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

test/setup-tests-beforeEach.ts:
```
// ==== VITEST ====
import { beforeEach} from 'vitest'

const NLE = process.env.npm_lifecycle_event

beforeEach(() => {
    process.env.npm_lifecycle_event = NLE
})
```




### Define variables in setupFiles and use them in child tests
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

</details>
























<br><br>
<br><br>

## globalSetup
- https://vitest.dev/config/#globalsetup
- **Notice that this file should be used to start an express server or something like this. If you want to declare variables before each tests you should use setupFiles instead. Because if you would set global.test=123 in the globalSetup file in the exported setup() then it would be undefined in your tests.**


<details><summary>Click to expand..</summary>
  
test/setup-tests.ts:
```typescript
export async function setup() {
    server.start()
}

export async function teardown() {
    server.close()
}
```
  - A global setup file can either export named functions setup and teardown or a default function that returns a teardown function ([example](https://github.com/vitest-dev/vitest/blob/main/test/global-setup/vitest.config.ts)).



</details>







<br><br>
<br><br>
<br><br>
<br><br>


## watch
- disable watch


































<br><br>
<br><br>
___________________________________
___________________________________
<br><br>
<br><br>

# Snapshot
- https://vitest.dev/guide/snapshot
```typescript
import { expect, it } from 'vitest'

it('toUpperCase', () => {
  const result = toUpperCase('foobar')
  expect(result).toMatchSnapshot()
})
```
he first time this test is run, Vitest creates a snapshot file that looks like this:
```typescript
// Vitest Snapshot v1, https://vitest.dev/guide/snapshot.html
exports['toUpperCase 1'] = '"FOOBAR"'
```
The snapshot artifact should be committed alongside code changes, and reviewed as part of your code review process. On subsequent test runs, Vitest will compare the rendered output with the previous snapshot. If they match, the test will pass. If they don't match, either the test runner found a bug in your code that should be fixed, or the implementation has changed and the snapshot needs to be updated.































<br><br>
<br><br>
___________________________________
___________________________________
<br><br>
<br><br>

# Tests

<br><br>
<br><br>


## Test Environment
<details><summary>Click to expand..</summary>
- https://vitest.dev/guide/environment.html#test-environment
- Vitest provides environment option to run code inside a specific environment. You can modify how environment behaves with environmentOptions option.

By default, you can use these environments:

    node is default environment
    jsdom emulates browser environment by providing Browser API, uses jsdom package
    happy-dom emulates browser environment by providing Browser API, and considered to be faster than jsdom, but lacks some API, uses happy-dom package
    edge-runtime emulates Vercel's edge-runtime, uses @edge-runtime/vm package


<br><br>

### populateGlobal
- **Notice that this may f up with your tests in some cases when you use glob. Not sure why**

setupFiles:
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

test.ts:
```typescript
test('sollte auf globale Variable zugreifen', () => {
  expect(global.modelDetails.test).toBe(1234);
});
```


</details>

































<br><br>
<br><br>
___________________________________
___________________________________
<br><br>
<br><br>

























<br><br>
<br><br>

## Nested Tests
- If you want to run e.g. pretest.ts before main.test.ts then do:

pretest.ts
```typescript
// ==== VITEST ====
import { beforeAll } from 'vitest'

beforeAll(() => {
    // ..
})
```


main.test.ts

Alternative #1
```typescript
// ==== VITEST ====
import { beforeAll } from 'vitest'

describe('[INTEGRATION] - src/errors/BaseError', () => {
    beforeAll(async() => {
        await import('./pretest')
    })

    it('should return 500 with BaseError details - error passed', async() => {
        // ..
    })
})
```

Alternative #2
```typescript
await import('./pretest')

describe('[INTEGRATION] - src/errors/BaseError', () => {
    it('should return 500 with BaseError details - error passed', async() => {
        // ..
    })
})
```








<br><br>
<br><br>
<br><br>
<br><br>

## Test Filtering
- https://vitest.dev/guide/filtering.html
  
<br><br>

### .only
- https://vitest.dev/guide/filtering#selecting-suites-and-tests-to-run


  
- In the latest version is should work as expected and only the selecterd test should run and not all other parallel next to it. If not here are some dirty workarounds:


<details><summary>Click to expand..</summary>
  
# test-only.sh 
```shell
grep --exclude-dir=node_modules -rl . -e 'test.only\|it.only\|describe.only' --null | tr '\n' ' ' | xargs -0 npx vitest | grep . || npx vitest --coverage
```

# test-only.ps1 (Windows - Powershell)
 - This solution actually creates a new config with only the file..
```powershell
# PowerShell-Äquivalent zum Linux-Skript (hochoptimiert für Geschwindigkeit):
# grep --exclude-dir=node_modules -rl . -e 'test.only\|it.only\|describe.only' --null | tr '\n' ' ' | xargs -0 npx vitest --typecheck --testTimeout=300000 --watch=false --disable-console-intercept | grep . || npx vitest --typecheck --coverage --watch=false --testTimeout=300000 --disable-console-intercept

# Verwende Select-String direkt mit Ausschluss von node_modules für maximale Geschwindigkeit
$foundFiles = Get-ChildItem -Recurse -File -Include "*.ts","*.js","*.tsx","*.jsx" | 
    Where-Object { $_.FullName -notlike "*\node_modules\*" } |
    Select-String -Pattern "test\.only|it\.only|describe\.only" -List |
    Select-Object -ExpandProperty Path -Unique

if ($foundFiles -and $foundFiles.Count -gt 0) {
    Write-Host "Gefundene .only Tests in:" -ForegroundColor Cyan
    $foundFiles | ForEach-Object { Write-Host "  $_" -ForegroundColor Green }
    
    # Erstelle eine temporäre Vitest-Konfiguration, die nur die gefundenen Dateien testet
    $tempConfigPath = "vitest.only.config.ts"
    $relativePaths = $foundFiles | ForEach-Object { 
        $path = $_ -replace [regex]::Escape((Get-Location).Path + "\"), ""
        $path = $path -replace "\\", "/"
        "`"$path`""
    }
    
    $configContent = @"
import { defineConfig } from 'vitest/config'
import { fileURLToPath } from 'node:url'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [tsconfigPaths()],
  test: {
    include: [
      $($relativePaths -join ",`n      ")
    ],
    testTimeout: 300000,
    typecheck: true,
    threads: false
  }
})
"@
    
    Set-Content -Path $tempConfigPath -Value $configContent
    
    # Führe Vitest mit der temporären Konfiguration aus
    npx vitest run --config $tempConfigPath --disable-console-intercept
    
    # Prüfe den Exit-Code von Vitest
    if ($LASTEXITCODE -eq 0) {
        # Lösche die temporäre Konfiguration
        Remove-Item -Path $tempConfigPath -Force
        # Tests wurden erfolgreich ausgeführt
        exit 0
    } else {
        # Lösche die temporäre Konfiguration
        Remove-Item -Path $tempConfigPath -Force
        Write-Host "Die .only Tests waren nicht erfolgreich, führe alle Tests aus..." -ForegroundColor Yellow
    }
} else {
    Write-Host "Keine .only Tests gefunden, führe alle Tests aus..." -ForegroundColor Yellow
}

# Führe alle Tests aus
npx vitest --typecheck --coverage --watch=false --disable-console-intercept 
```

</details>












<br><br>
<br><br>

## Coverage
- https://vitest.dev/guide/coverage.html#coverage-setup




















<br><br>
<br><br>
___________________________________
___________________________________
<br><br>
<br><br>

# API





<br><br>
<br><br>

## Testing Types
- https://vitest.dev/guide/testing-types
- Vitest allows you to write tests for your types, using expectTypeOf or assertType syntaxes. By default all tests inside *.test-d.ts files are considered type tests, but you can change it with typecheck.include config option.

<details><summary>Click to expand..</summary>

- Here is an example to type check any .test.ts file instead of only *.test-d.ts. **However, it is recommended to create own tests files for type checks instead of bundle everything into as single file**
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

- Under the hood Vitest calls tsc or vue-tsc, depending on your config, and parses results. Vitest will also print out type errors in your source code, if it finds any. You can disable it with typecheck.ignoreSourceErrors config option.Keep in mind that Vitest doesn't run these files, they are only statically analyzed by the compiler. Meaning, that if you use a dynamic name or test.each or test.for, the test name will not be evaluated - it will be displayed as is.

- To enable typechecking, just add --typecheck flag to your Vitest command in package.json:
```
{
  "scripts": {
    "test": "vitest --typecheck"
  }
}
```


</details>












<br><br><br><br>
<br><br><br><br>

## VI Utility
- https://vitest.dev/api/vi.html
- Vitest provides utility functions to help you out through its vi helper. You can access it globally (when globals configuration is enabled), or import it from vitest directly:
```typescript
import { vi } from 'vitest'
```
  
<details><summary>Click to expand..</summary>

# vi.stubEnv 
- Changes the value of environmental variable on process.env and import.meta.env. You can restore its value by calling vi.unstubAllEnvs.
```typescript
import { vi } from 'vitest'

// `process.env.NODE_ENV` and `import.meta.env.NODE_ENV`
// are "development" before calling "vi.stubEnv"

vi.stubEnv('NODE_ENV', 'production')

process.env.NODE_ENV === 'production'
import.meta.env.NODE_ENV === 'production'

vi.stubEnv('NODE_ENV', undefined)

process.env.NODE_ENV === undefined
import.meta.env.NODE_ENV === undefined

// doesn't change other envs
import.meta.env.MODE === 'development'
```

<br><br>

# vi.unstubAllEnvs 
- https://vitest.dev/api/vi.html#vi-unstuballenvs
- Restores all import.meta.env and process.env values that were changed with vi.stubEnv. When it's called for the first time, Vitest remembers the original value and will store it, until unstubAllEnvs is called again.
```typescript
import { vi } from 'vitest'

// `process.env.NODE_ENV` and `import.meta.env.NODE_ENV`
// are "development" before calling "vi.stubEnv"

vi.stubEnv('NODE_ENV', 'production')

process.env.NODE_ENV === 'production'
import.meta.env.NODE_ENV === 'production'

vi.stubEnv('NODE_ENV', undefined)

process.env.NODE_ENV === undefined
import.meta.env.NODE_ENV === undefined

// doesn't change other envs
import.meta.env.MODE === 'development'
```
 






 
</details>
 
<br><br>




















<br><br><br><br>
<br><br><br><br>

## expectTypeOf
- https://vitest.dev/api/expect-typeof
  
<details><summary>Click to expand..</summary>


# private class constructor
- As far as I know there is no way to test a private constructor for params types. You can only create a new method like below. However, when you use getInstance as Single method and this is the reason why your constructor is private the you can test this method:
```typescript
class Foo {
    private constructor(private x: number) {}

    /** @internal */
    static createForTesting(x: number) {
        return new Foo(x);
    }
}

const instance = Foo.createForTesting(5);
```

<br><br>

# toEqualTypeOf 
```typescript
expectTypeOf({ a: 1 }).toEqualTypeOf<{ a: number }>()
expectTypeOf({ a: 1 }).toEqualTypeOf({ a: 1 })
expectTypeOf({ a: 1 }).toEqualTypeOf({ a: 2 })
expectTypeOf({ a: 1, b: 1 }).not.toEqualTypeOf<{ a: number }>()
```

<br><br>

## Generics
```typescript
expectTypeOf(modelManager.createModel<TMongooseSchema>).parameter(0).toEqualTypeOf<IModelCore<TMongooseSchema>>()

  /**
     * Creates a Mongoose model based on the given name, schema, and database name.
     * @template TMongooseSchema - The type of the mongoose schema.
     * @param modelDetails - An object containing the model's details.
     * @returns A promise that resolves to the created Mongoose Model instance.
     */
    public async createModel<TMongooseSchema>({
        modelName,
        schema,
        dbName
    }: IModelCore<TMongooseSchema>): Promise< mongoose.Model<TMongooseSchema> > {
        const mongooseUtils = await MongooseUtils.getInstance(dbName)
        const Model = await mongooseUtils.createModel<TMongooseSchema>(schema, modelName)
        
        // Ensure indexes are created for the model
        await Model.createIndexes()
        return Model
    }
```

<br><br>
<br><br>

# parameter
- https://vitest.dev/api/expect-typeof.html#parameter
```typescript
expectTypeOf(modelManager['globModels']).parameter(0).toBeString()
expectTypeOf(modelManager.createModel<TMongooseSchema>).parameter(0).toEqualTypeOf<IModelCore<TMongooseSchema>>()
```
- **This will not work with static methods in classes. Instead use .toBeCallableWith()


<br><br>
<br><br>

# toBeCallableWith
- https://vitest.dev/api/expect-typeof.html#tobecallablewith
```typescript
expectTypeOf(ModelUtils.createMemoryModel).toBeCallableWith(modelCoreDetails)
```



<br><br>
<br><br>

# returns 
- You can use .returns to extract return value of a function type.
```typescript
import { expectTypeOf } from 'vitest'

expectTypeOf(() => {}).returns.toBeVoid()
expectTypeOf((a: number) => [a, a]).returns.toEqualTypeOf([1, 2])
expectTypeOf(modelManager['getModels']).returns.toEqualTypeOf<IModel<mongoose.SchemaDefinition<{}>>[]>()

# If needed you can call a promise by yourself and check it with returns but it is recommended to use resolve
expectTypeOf(ModelManager.getInstance).returns.toEqualTypeOf(Promise.resolve(modelManager))
```


<br><br>

# resolve 
- This matcher extracts resolved value of a Promise, so you can perform other assertions on it.
```typescript
import { expectTypeOf } from 'vitest'

async function asyncFunc() {
  return 123
}

expectTypeOf(asyncFunc).returns.resolves.toBeNumber()
expectTypeOf(Promise.resolve('string')).resolves.toBeString())

# OTHER EXAMPLES
expectTypeOf(ModelManager.getInstance).returns.resolves.toEqualTypeOf<ModelManager>()

# Private class methods
expectTypeOf(modelManager['globModels']).returns.resolves.toEqualTypeOf<IModel<mongoose.SchemaDefinition<{}>>[]>()
expectTypeOf(modelManager['init']).returns.resolves.toBeVoid()
```




<br><br>

# instance 
- https://vitest.dev/api/expect-typeof#instance
- This property gives access to matchers that can be performed on an instance of the provided class.
```typescript
import { expectTypeOf } from 'vitest'

expectTypeOf(Date).instance.toHaveProperty('toISOString')
```



</details>





















<br><br>
<br><br>
<br><br>
<br><br>

## expect
- https://vitest.dev/api/expect.html

<details><summary>Click to expand..</summary>
 
<br><br>
<br><br>

# objectContaining
- https://vitest.dev/api/expect.html#expect-objectcontaining
```javascript
import { expect, test } from 'vitest'

# Example #1
expect(chatCompletion).toEqual(expect.objectContaining(expectedResponse))

# Example #2
test('basket has empire apples', () => {
  const basket = {
    varieties: [
      {
        name: 'Empire',
        count: 1,
      }
    ],
  }
  expect(basket).toEqual({
    varieties: [
      expect.objectContaining({ name: 'Empire' }),
    ]
  })
})
```

<br><br>
<br><br>

# toThrowError()
- https://vitest.dev/api/expect.html#tothrowerror

<br><br>

## Sync
```javascript
import { expect, test } from 'vitest'

function getFruitStock(type: string) {
  if (type === 'pineapples') {
    throw new Error('Pineapples are not in stock')
  }

  // Do some other stuff
}

test('throws on pineapples', () => {
  // Test that the error message says "stock" somewhere: these are equivalent
  expect(() => getFruitStock('pineapples')).toThrowError(/stock/)
  expect(() => getFruitStock('pineapples')).toThrowError('stock')

  // Test the exact error message
  expect(() => getFruitStock('pineapples')).toThrowError(
    /^Pineapples are not in stock$/,
  )
})
```


## Async
```javascript
function getAsyncFruitStock() {
  return Promise.reject(new Error('empty'))
}

test('throws on pineapples', async () => {
  await expect(() => getAsyncFruitStock()).rejects.toThrowError('empty')
})
```

</details>
