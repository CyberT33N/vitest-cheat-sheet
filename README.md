# vitest-cheat-sheet


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
  rep --exclude-dir=node_modules -rl . -e 'test.only\|it.only\|describe.only' --null | tr '\n' ' ' | xargs -0 npx vitest | grep . || npx vitest --coverage
  ```













<br><br>
<br><br>

## Coverage
- https://vitest.dev/guide/coverage.html#coverage-setup
