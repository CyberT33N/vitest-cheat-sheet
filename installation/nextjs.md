# Next.js Installation

```shell
npm install -D vitest @vitejs/plugin-react jsdom @testing-library/react vite-tsconfig-paths @vitest/coverage-v8
```

## Konfiguration

Erstelle eine `vitest.config.ts` Datei mit folgendem Inhalt:

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