# Test Filtering

Vitest bietet verschiedene Möglichkeiten, Tests zu filtern und nur bestimmte Tests auszuführen.

Offizielle Dokumentation: [https://vitest.dev/guide/filtering.html](https://vitest.dev/guide/filtering.html)

## .only

<details><summary>Click to expand..</summary>

Mit `.only` können bestimmte Tests oder Testsuiten ausgewählt werden, die ausgeführt werden sollen.

Offizielle Dokumentation: [https://vitest.dev/guide/filtering#selecting-suites-and-tests-to-run](https://vitest.dev/guide/filtering#selecting-suites-and-tests-to-run)



```ts
test.only('macht dies und das', () => {
  // Testinhalt
})
```

Oder auf `describe`-Ebene:

```ts
describe.only('Gruppe von Tests', () => {
  test('macht A', () => {})
})
```






In der aktuellen Version sollte dies wie erwartet funktionieren und nur der ausgewählte Test sollte ausgeführt werden, nicht alle anderen parallel dazu. Falls nicht, gibt es hier einige Workarounds:

### Script für Linux (Bash)

```shell
grep --exclude-dir=node_modules -rl . -e 'test.only\|it.only\|describe.only' --null | tr '\n' ' ' | xargs -0 npx vitest | grep . || npx vitest --coverage
```

### Script für Windows (PowerShell)

Diese Lösung erstellt eine neue Konfiguration mit nur den Dateien, die `.only` enthalten:

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





---

### 🔹 2. CLI-Filter via Pattern Matching

```bash
vitest run -t "macht dies und das"
```

**`-t` bzw. `--testNamePattern`**: Führt nur Tests aus, deren Namen auf das Pattern passen. Regex-kompatibel.

```bash
vitest run -t "/macht.*das/"
```

Auch im **Debug-Modus** kombinierbar:

```bash
vitest run --inspect --run --threads=false -t "Testname"
```

---

### 🔹 3. Datei direkt ausführen

```bash
vitest run path/to/file.spec.ts
```

Optional mit `--inspect` zum Debuggen:

```bash
node --inspect-brk node_modules/vitest/vitest.mjs run path/to/file.spec.ts
```

Oder Shorty via:

```bash
vitest run --inspect path/to/file.spec.ts
```

---

### 🔹 4. VS Code Debug-Config (🔥 der elegante Weg)

`launch.json`:

```json
{
  "type": "node",
  "request": "launch",
  "name": "Debug Vitest",
  "program": "${workspaceFolder}/node_modules/vitest/vitest.mjs",
  "args": ["run", "--inspect-brk", "--threads=false", "tests/mein.test.ts"],
  "autoAttachChildProcesses": true,
  "smartStep": true,
  "skipFiles": ["<node_internals>/**"],
  "console": "integratedTerminal"
}
```

Du kannst `"args"` mit `-t "testname"` kombinieren für granulare Auswahl.

---

### 🔹 5. `vitest watch` mit interaktivem UI

Start:

```bash
vitest watch
```

Dann: Mit `p` (pattern) nur bestimmte Tests matchen.  
Mit `o` `.only`-Modi toggeln.  
Mit `t` Tests selektiv ausführen.

Cool zum Rumspielen – nix für CI, aber top fürs Debuggen.

---

### 🔹 6. Temporäres `.skip()` für alles andere

Nicht schön, aber brutal direkt:

```ts
test.skip('nicht jetzt', () => {})
```

Oder ALLES außer eins:

```ts
describe.skip('riesige Test-Gruppe', () => {
  // massiver Wust
})
```

---

### Bonus: 💡 Debug mit `--threads=false`

Das **deaktiviert parallele Threads**, wichtig für Debugger wie Chrome DevTools oder VS Code, sonst breakpoints werden ignoriert.

```bash
vitest run --inspect-brk --threads=false
```

---

### TL;DR (wie Kollegah’s Hook):

| Methode             | Nutzen                            | Debug-fähig |
|---------------------|-----------------------------------|-------------|
| `.only()`           | Schnell und im Code direkt        | ✅           |
| `-t`                | Präziser Name-Match               | ✅           |
| Dateipfad           | Nur bestimmte Datei               | ✅           |
| VS Code Config      | Sauber und wiederverwendbar       | ✅✅         |
| `vitest watch`      | Interaktives UI                   | ⚠️           |
| `.skip()`           | Holzhammer                        | ❌           |

---

Wenn du willst, baue ich dir `vitest-debug` alias-Skripte für `package.json`, damit du per Tastenkürzel alles steuern kannst wie ein Boss 😎.
