# vitest-cheat-sheet


# Install

## Next.js
```shell
npm install -D vitest @vitejs/plugin-react jsdom @testing-library/react vite-tsconfig-paths
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
