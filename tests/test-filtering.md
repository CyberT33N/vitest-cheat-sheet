# Test Filtering

Vitest bietet verschiedene Möglichkeiten, Tests zu filtern und nur bestimmte Tests auszuführen.

Offizielle Dokumentation: [https://vitest.dev/guide/filtering.html](https://vitest.dev/guide/filtering.html)


<br><br>


# 🔍 Coverage

<details><summary>Click to expand...</summary>


# Run specific test and only coverage for this test
```shell
npx vitest run test/unit/services/evident/EvidentService/abbreviation.test.ts --coverage --coverage.include="src/main/services/evident/EvidentService/abbreviation.ts"
```

</details>





<br><br>






## 🔍 Testausführung & Filter (ohne Debugging)

<details><summary>Click to expand...</summary>

### 📁 Einzelne Datei ausführen

```bash
npx vitest run path/to/your/test-file.test.ts
```

✅ Führt **nur diese eine Datei** aus  
✅ Ideal für gezielte Test-Sessions ohne `test.only`  
✅ Funktioniert auch mit Globs:

```bash
npx vitest run "tests/unit/**/*.test.ts"
```

---

### 🎯 Test nach Name filtern

```bash
npx vitest run -t "spezifischer Testname"
```

Oder mit Regex:

```bash
npx vitest run -t "/RegEx.*Pattern/"
```

✅ Führt **nur Tests aus**, deren Name exakt oder per Regex matcht  
✅ Kombinierbar mit Datei:

```bash
npx vitest run path/to/file.test.ts -t "spezifischer Test"
```

---

### 🔍 Tests per CLI nur bestimmte Suiten laufen lassen

Mit `--include`:

```bash
npx vitest run --include src/components/Button/*.test.ts
```

Oder mehrere:

```bash
npx vitest run --include "tests/unit/**/*.test.ts" "tests/integration/**/*.test.ts"
```

---

### ⏳ Timeout setzen

```bash
npx vitest run --testTimeout=30000
```

Setzt Timeout für alle Tests auf 30 Sekunden – kein Warten auf Zombies 🧟‍♂️

---

### 🧪 Typechecking + Coverage in einem Rutsch

```bash
npx vitest run --typecheck --coverage
```

✅ Type Safety  
✅ Test Coverage  
✅ Kein Debug-Modus, einfach durchlaufen lassen

---

### ⚡ Mehr Speed (ohne Threads = sequentiell)

```bash
npx vitest run --threads=false
```

Perfekt für flaky Tests, race conditions, oder wenn du CI/CD-Ticks debuggen willst (aber ohne richtigen Debugger).

---

### 🧠 Pro-Tipp: Custom Scripts

```json
"scripts": {
  "test:file": "vitest run path/to/file.test.ts",
  "test:unit": "vitest run tests/unit",
  "test:name": "vitest run -t 'mein testname'"
}
```

Dann einfach:

```bash
npm run test:file
```

oder via VS Code „Debug Script“.

---

### 🔚 TL;DR

| Use Case                     | Befehl                                                                 |
|-----------------------------|------------------------------------------------------------------------|
| Einzeldatei                 | `vitest run path/to/file.test.ts`                                      |
| Test nach Name              | `vitest run -t "testname"`                                             |
| Datei + Testname kombinieren | `vitest run path/to/file -t "testname"`                                |
| Typecheck + Coverage        | `vitest run --typecheck --coverage`                                   |
| Ohne Parallelisierung       | `vitest run --threads=false`                                          |

---

💥 Damit testest du gezielt, präzise und ohne `.only` – wie ein Scharfschütze im Test-Dschungel 🥷

</details>

---




























<br><br>
---
<br><br>


# Debugging
- **Am einfachsten ist es, die **Vitest** VSCode-Extension zu benutzen. Dort kannst du dann einfach Tests einzeln starten, indem du z.B. mit der rechten Maustaste auf den Play-Button drückst und dann **Debug-Test** klickst. Wenn du die VSCode-Extension nicht benutzen **WILLST**, hast du nur die folgenden **MÖGLICHKEITEN** unten.**


<details><summary>Click to expand..</summary>



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
# Finde rekursiv alle Dateien im 'test'-Verzeichnis, die 'test.only', 'it.only' oder 'describe.only' enthalten.
$onlyFiles = Get-ChildItem -Path ./test -Recurse | Where-Object { !$_.PSIsContainer } | Select-String -Pattern 'test\.only|it\.only|describe\.only' -List | ForEach-Object { $_.Path }

# Prüfe, ob Dateien mit '.only' gefunden wurden.
if ($onlyFiles.Count -gt 0) {
    Write-Host "⚠️ Found .only tests in:"
    # Gib die gefundenen Dateien aus (optional, aber hilfreich für die Fehlersuche).
    $onlyFiles | ForEach-Object { Write-Host "- $_" }
    Write-Host "🚀 Running vitest only on these files..."
    # Führe vitest NUR für die gefundenen Dateien aus.
    # PowerShell übergibt die Array-Elemente als separate Argumente an npx.
    npx vitest $onlyFiles --typecheck --coverage --watch=false --disable-console-intercept
}
```




</details>






---

### 🧠 Warum das so ist:

Wenn du `vitest` im **Terminal** ausführst mit `--inspect-brk`,  
dann *läuft der Node-Prozess zwar im Debug-Modus*,  
aber **VS Code weiß nix davon** — kein Attach, kein Magic, keine Breakpoints 💥

---

### 💣 Das Terminal ≠ Debug-Konsole

VS Code erkennt nur Debug-Sessions, wenn:

1. Du sie über `launch.json` startest  
2. Du ein Skript aus `package.json` über **"Debug Script"** startest  
3. Oder du ein **Attach-Profil** manuell aktivierst

---

### ✅ Drei Lösungen, um das sauber zu machen:

---

#### 🔹 **Lösung 1: Nutze `launch.json` für alles**

Mach dir mehrere Einträge:

```json
{
  "type": "node",
  "request": "launch",
  "name": "Debug Vitest File",
  "program": "${workspaceFolder}/node_modules/vitest/vitest.mjs",
  "args": ["run", "--inspect-brk", "--no-file-parallelism", "test/unit/services/evident/EvidentDatabaseIsolation.test.ts"],
  "console": "integratedTerminal",
  "autoAttachChildProcesses": true
}
```

Oder mit `-t` für gezielten Test:

```json
"args": ["run", "--inspect-brk", "--no-file-parallelism", "-t", "spezifischer Testname"]
```

Dann per F5 starten oder in der Debug-Leiste auswählen – und **Breakpoints wirken wie Zauber** ✨

---

#### 🔹 **Lösung 2: Run-Skripte debuggen (dein Weg mit Hover)**

In `package.json`:

```json
"scripts": {
  "test:debug": "vitest run --inspect-brk --no-file-parallelism test/unit/services/evident/EvidentDatabaseIsolation.test.ts"
}
```

Dann:
- Hover über das Skript im `package.json`
- Klick auf **"Debug Script"**

✅ Breakpoints feuern  
✅ Kein extra `launch.json` nötig  
✅ Shortcuts wie `STRG + SHIFT + P → Debug npm script` funktionieren

---

#### 🔹 **Lösung 3: Attach to Running Process (manual gangsta mode)**

Wenn du **unbedingt aus dem Terminal** starten willst:

```bash
vitest run --inspect-brk --no-file-parallelism test/unit/services/evident/EvidentDatabaseIsolation.test.ts
```

Dann in VS Code:

- `STRG + SHIFT + P` → `Debug: Attach to Node Process`
- Pick den richtigen PID

⚠️ Klappt, aber ist nerviger als direkt F5

---

### ✅ Fazit:  
Debugging in Node ist kein Hexenwerk, aber VS Code muss **explizit wissen**, dass er attachen soll. Nur dann setzt er die Breakpoints korrekt.

---

Willst du eine `launch.json` mit verschiedenen Targets (Einzeltest, Pattern, Datei)? Ich bau dir die wie ein Maschinengewehr mit verschiedenen Feuermodi.



</details>

