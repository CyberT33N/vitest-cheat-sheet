# Vitest cheat sheet








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

### vitest.unit.config.ts
- package.json add to scripts `"test:unit": "vitest --config vitest.unit.config.ts"`

```
import { defineConfig } from 'vitest/config'
import vitestConfig from './vitest.config'

const config = {
    ...vitestConfig,
    test: vitestConfig.test || {}
}

config.test.include = [
    'test/unit/**/*.test.ts'
]

config.test.watch = false

export default defineConfig(config)
```





<br><br>
<br><br>
<br><br>
<br><br>

## globalSetup
- test/setup-tests.ts:
```typescript
// ==== BOOTSTRAP ====
import { bootstrap } from '@/src/bootstrap'
export default bootstrap
```
  - export function which will be executed once before all tests


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

## expect
- https://vitest.dev/api/expect.html

<br><br>
<br><br>

### objectContaining
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

### toThrowError()
- https://vitest.dev/api/expect.html#tothrowerror

<br><br>

#### Sync
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


#### Async
```javascript
function getAsyncFruitStock() {
  return Promise.reject(new Error('empty'))
}

test('throws on pineapples', async () => {
  await expect(() => getAsyncFruitStock()).rejects.toThrowError('empty')
})
```
