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

### 🔹 2. CLI-Filter via Pattern Matching (mit Debug)

```bash
vitest run -t "macht dies und das"
```

**`-t` bzw. `--testNamePattern`**: Führt nur Tests aus, deren Namen auf das Pattern passen. Regex geht auch.

```bash
vitest run -t "/macht.*das/"
```

👉 **Debug-Modus richtig**:

```bash
vitest run --inspect-brk --no-file-parallelism -t "Testname"
```

---

### 🔹 3. Datei direkt ausführen (Debug-safe)

```bash
vitest run path/to/file.spec.ts
```

✅ Debug-Version:

```bash
vitest run --inspect-brk --no-file-parallelism path/to/file.spec.ts
```

Oder komplett ohne vitest bin-wrapper:

```bash
node --inspect-brk node_modules/vitest/vitest.mjs run --no-file-parallelism path/to/file.spec.ts
```

---

### 🔹 4. VS Code Debug-Config (🔥 elegant und funktional)

```json
{
  "type": "node",
  "request": "launch",
  "name": "Debug Vitest File",
  "program": "${workspaceFolder}/node_modules/vitest/vitest.mjs",
  "args": ["run", "--inspect-brk", "--no-file-parallelism", "tests/mein.test.ts"],
  "autoAttachChildProcesses": true,
  "smartStep": true,
  "skipFiles": ["<node_internals>/**"],
  "console": "integratedTerminal"
}
```

➡️ Kombinierbar mit:

```json
"args": ["run", "--inspect-brk", "--no-file-parallelism", "-t", "spezifischer Test"]
```

---

### 🔹 5. `vitest watch` mit interaktivem UI

```bash
vitest watch
```

Nutze:
- `p` → Pattern-Filter (Testnamen)
- `o` → `.only` toggle
- `t` → einzelne Tests ausführen

🧠 Tipp: Kein echter Debug-Modus. Kein `--inspect-brk` hier – nur gut zum schnellen Rumspielen.

---

### 🔹 6. Temporäres `.skip()` für radikale Fokussierung

```ts
test.skip('nicht jetzt', () => {})
```

Oder Gruppenweise:

```ts
describe.skip('nicht heute', () => {})
```

---

### 💡 Debug richtig = `--no-file-parallelism`

Das ist **Pflicht**, wenn du mit `--inspect-brk` arbeitest. Alles andere kracht.

```bash
vitest run --inspect-brk --no-file-parallelism
```

---

### ⚡ TL;DR (wie Kollegah’s Hook nach 8 Lines Aufbau):

| Methode             | Nutzen                            | Debug-fähig           |
|---------------------|-----------------------------------|------------------------|
| `.only()`           | Schnell, direkt im Code           | ✅                     |
| `-t`                | Präziser Testname-Match           | ✅ (mit `--no-file-parallelism`) |
| Dateipfad           | Nur bestimmte Datei               | ✅ (dito)              |
| VS Code Config      | Reproduzierbar & bequem           | ✅✅                   |
| `vitest watch`      | Interaktiv, aber ohne Debugger    | ⚠️                     |
| `.skip()`           | Holzhammer                        | ❌                     |

---

Wenn du willst, baller ich dir noch `npm run` Shortcuts rein:

```json
"scripts": {
  "test:debug:file": "vitest run --inspect-brk --no-file-parallelism",
  "test:debug:name": "vitest run --inspect-brk --no-file-parallelism -t"
}
```

Dann einfach:

```bash
npm run test:debug:file -- tests/mein.test.ts
npm run test:debug:name -- "mein Testname"
```

Jetzt bist du vollgepanzert. Debuggen wie ein Sniper auf Speed 😎
