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
























<br><br>
<br><br>
___________________________________
___________________________________
<br><br>
<br><br>

# vitest.config.ts
- https://vitest.dev/config/

<br><br>

## Load environment varialbes
```typescript
// ==== VITEST ====
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import tsconfigPaths from 'vite-tsconfig-paths'

// ==== NEXT.JS ====
import { loadEnvConfig } from '@next/env'
loadEnvConfig(process.cwd())

export default defineConfig({
    plugins: [tsconfigPaths(), react()],
    test: {
        environment: 'node',
        globalSetup: 'test/setup-tests.ts'
    }
})
```
- environment must be node







<br><br>
<br><br>
<br><br>
<br><br>

## vitest.unit.config.ts
- package.json add to scripts `"test:unit": "vitest run --testTimeout=300000 --coverage --disable-console-intercept --watch=false --config vitest.unit.config.ts"`

```typescriot
// ==== VITEST ====
import { defineConfig, mergeConfig } from 'vitest/config'
import vitestConfig from './vitest.config'

export default mergeConfig(vitestConfig, defineConfig({
    test: {
        include: ['test/unit/**/*.test.ts'],
        setupFiles: 'test/unit/pretestEach.ts',
        watch: false,
        coverage: {
            exclude: ['**/route.ts']
        }
    }
}))
```

<br><br>
<br><br>

## vitest.integration.config.ts
- package.json add to scripts `"test:integration": "vitest run --testTimeout=300000 --coverage --disable-console-intercept --watch=false --config vitest.integration.config.ts"`

```typescriot

// ==== VITEST ====
import { defineConfig, mergeConfig } from 'vitest/config'
import vitestConfig from './vitest.config'

const cfg = mergeConfig(vitestConfig, defineConfig({
    test: {
        include: ['test/integration/**/*.test.ts'],
        globalSetup: 'test/integration/pretestAll.ts',
        watch: false
    }
}))


cfg.test.coverage.include = ['src/middleware.ts']

export default cfg
```

















<br><br>
<br><br>
<br><br>
<br><br>

## setupFiles
- https://vitest.dev/config/#setupfiles
- Path to setup files. They will be run before each test file.

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







<br><br>
<br><br>

## globalSetup
- https://vitest.dev/config/#globalsetup
- test/setup-tests.ts:
```typescript
export async function setup() {
    server.start()
}

export async function teardown() {
    server.close()
}
```
  - A global setup file can either export named functions setup and teardown or a default function that returns a teardown function ([example](https://github.com/vitest-dev/vitest/blob/main/test/global-setup/vitest.config.ts)).









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

# Tests




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
- Not sure how it works out of the box but here is a workaround:

  - test-only.sh 
  ```shell
  grep --exclude-dir=node_modules -rl . -e 'test.only\|it.only\|describe.only' --null | tr '\n' ' ' | xargs -0 npx vitest | grep . || npx vitest --coverage
  ```













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

## Testing Types
- https://vitest.dev/guide/testing-types
- Vitest allows you to write tests for your types, using expectTypeOf or assertType syntaxes. By default all tests inside *.test-d.ts files are considered type tests, but you can change it with typecheck.include config option. Here is an example to type check any .test.ts file instead of only *.test-d.ts
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

<br><br>

# toEqualTypeOf 
```typescript
expectTypeOf({ a: 1 }).toEqualTypeOf<{ a: number }>()
expectTypeOf({ a: 1 }).toEqualTypeOf({ a: 1 })
expectTypeOf({ a: 1 }).toEqualTypeOf({ a: 2 })
expectTypeOf({ a: 1, b: 1 }).not.toEqualTypeOf<{ a: number }>()
```


<br><br>

# returns 
- You can use .returns to extract return value of a function type.
```typescript
import { expectTypeOf } from 'vitest'

expectTypeOf(() => {}).returns.toBeVoid()
expectTypeOf((a: number) => [a, a]).returns.toEqualTypeOf([1, 2])
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
