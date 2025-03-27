## 🧪 Vitest `vi` Helper Cheatsheet

**Globale Verfügbarkeit:** `vi` ist global verfügbar (wenn `globals: true` in Vitest Konfiguration) oder importierbar:

```typescript
import { vi } from 'vitest'
```

---

### 📦 Mock Module

* **`vi.mock(path, factory?)`** 📍
    * Ersetzt importierte Module unter `path` mit einem Mock.
    * `path`: Modul-Pfad (Vite Aliase möglich).
    * `factory?`:
        * Funktion, die das Mock-Modul zurückgibt (einmalig aufgerufen, Ergebnisse gecached).
        * Objekt mit `{ spy: true }` für Auto-Mocking ohne Implementierungsüberschreibung.
    * **Wichtig:** Funktioniert nur mit `import`, nicht `require()`. Hoisting!
    * `vi.mock` wird **gehoisted** (vor Imports ausgeführt). Für nicht-gehoisted: `vi.doMock`.
    * `__mocks__` Ordner neben Datei oder im Projekt-Root wird automatisch als Mock verwendet (falls `factory` fehlt).

    ```typescript
    vi.mock('./src/modul.ts', () => {
      return {
        namedExport: vi.fn(),
        default: { mockDefault: vi.fn() }
      }
    })

    vi.mock('./src/rechner.ts', { spy: true }) // Auto-Mocking, Original-Implementierung bleibt
    ```

* **`vi.doMock(path, factory?)`** 📍
    * Wie `vi.mock`, aber **nicht gehoisted**. Mocks nur *nachfolgende* Imports.
    * Nützlich, um Variablen aus dem globalen Scope im `factory` zu verwenden.

* **`vi.mocked<T>(obj, deep?)` / `vi.mocked<T>(obj, options?)`** ⌨️
    * TypeScript Type-Helper. Gibt `obj` zurück, informiert TypeScript über Mocking.
    * `deep: true`:  TypeScript nimmt an, dass das gesamte Objekt tief gemockt ist.
    * `partial: true`: Erwartet `Partial<T>` als Rückgabewert.

    ```typescript
    import * as example from './example'
    vi.mock('./example')
    vi.mocked(example.add).mockReturnValue(10)
    ```

* **`vi.importActual<T>(path)`** 📦
    * Importiert das **originale** Modul, **ohne Mocking**.
    * Für partielles Mocking nützlich.

    ```typescript
    vi.mock('./example.js', async () => {
      const originalModule = await vi.importActual('./example.js')
      return { ...originalModule, get: vi.fn() } // Original + Mock
    })
    ```

* **`vi.importMock<T>(path)`** 📦
    * Importiert ein Modul, bei dem **alle** Eigenschaften (auch verschachtelte) gemockt sind.
    * Folgt den gleichen Regeln wie `vi.mock`.

* **`vi.unmock(path)`** 📦
    * Entfernt Modul aus der Mock-Registry. Zukünftige `import` liefern das originale Modul.
    * **Gehoisted**. Betrifft Module aus `setupFiles`.

* **`vi.doUnmock(path)`** 📦
    * Wie `vi.unmock`, aber **nicht gehoisted**.  Nächster `import` liefert original, vorherige Imports bleiben gemockt.

* **`vi.resetModules()`** 🔄
    * Leert den Modul-Cache. Module werden bei Re-Import neu evaluiert.
    * **Achtung:**  Resettet **nicht** die Mock-Registry.

* **`vi.dynamicImportSettled()`** ⏳
    * Wartet, bis alle dynamischen Imports geladen sind.
    * Nützlich bei synchronen Aufrufen, die dynamische Imports starten.

---

### 🛠️ Mocking Funktionen und Objekte

* **`vi.fn(fn?)`** 🎭
    * Erstellt einen Mock-Spy für eine Funktion (optional mit Originalfunktion).
    * Speichert Aufrufargumente, Rückgabewerte, Instanzen. Verhalten manipulierbar mit Methoden (`mockReturnValueOnce`, etc.).

    ```typescript
    const mockFn = vi.fn(() => 'original return');
    mockFn.mockReturnValueOnce('mocked once');
    mockFn(); // Gibt 'mocked once' zurück
    mockFn(); // Gibt 'original return' zurück
    expect(mockFn).toHaveBeenCalled();
    ```

* **`vi.isMockFunction(fn)`** ✅
    * Prüft, ob eine Funktion eine Mock-Funktion ist (Type-Narrowing in TypeScript).

* **`vi.clearAllMocks()`** 🧹
    * Ruft `.mockClear()` auf allen Spies auf. Leert Mock-History, behält Implementierung bei.

* **`vi.resetAllMocks()`** ♻️
    * Ruft `.mockReset()` auf allen Spies auf. Leert Mock-History, setzt Implementierung auf Original zurück.

* **`vi.restoreAllMocks()`** 💯
    * Ruft `.mockRestore()` auf allen Spies auf. Leert Mock-History, stellt Original-Implementierungen und Deskriptoren wieder her.  Nützlich in `afterEach`.

* **`vi.spyOn<T, K extends keyof T>(object, method, accessType?)`** 👀
    * Erstellt einen Spy auf eine Methode oder Getter/Setter eines Objekts. Gibt eine Mock-Funktion zurück.

    ```typescript
    const obj = { method: () => 42 };
    const spy = vi.spyOn(obj, 'method').mockReturnValue(10);
    obj.method(); // Gibt 10 zurück
    vi.restoreAllMocks();
    obj.method(); // Gibt 42 zurück
    ```
    * **Achtung:** SpyOn auf exportierte Methoden im Browser-Modus evtl. nicht möglich. Alternative: `vi.mock("./file-path.js", { spy: true })`.

* **`vi.stubEnv(name, value)`** 🌍
    * Ändert Umgebungsvariablen in `process.env` und `import.meta.env`.
    * `value`: `string`, `undefined` oder `boolean` (für "PROD", "DEV", "SSR").

Ändert den Wert einer Umgebungsvariable in `process.env` und `import.meta.env`. Sie können den ursprünglichen Wert durch Aufruf von `vi.unstubAllEnvs` wiederherstellen.

```typescript
import { vi } from 'vitest'

// `process.env.NODE_ENV` und `import.meta.env.NODE_ENV`
// sind "development" vor dem Aufruf von "vi.stubEnv"

vi.stubEnv('NODE_ENV', 'production')

process.env.NODE_ENV === 'production'
import.meta.env.NODE_ENV === 'production'

vi.stubEnv('NODE_ENV', undefined)

process.env.NODE_ENV === undefined
import.meta.env.NODE_ENV === undefined

// ändert keine anderen Umgebungsvariablen
import.meta.env.MODE === 'development'
```


* **`vi.unstubAllEnvs()`** 🌍
    * Stellt alle mit `vi.stubEnv` geänderten Umgebungsvariablen auf ihre ursprünglichen Werte zurück.

Stellt alle `import.meta.env` und `process.env` Werte wieder her, die mit `vi.stubEnv` geändert wurden. Wenn es zum ersten Mal aufgerufen wird, merkt sich Vitest den Originalwert und speichert ihn, bis `unstubAllEnvs` erneut aufgerufen wird.

```typescript
import { vi } from 'vitest'

// `process.env.NODE_ENV` und `import.meta.env.NODE_ENV`
// sind "development" vor dem Aufruf von "vi.stubEnv"

vi.stubEnv('NODE_ENV', 'production')

process.env.NODE_ENV === 'production'
import.meta.env.NODE_ENV === 'production'

vi.unstubAllEnvs()

// Werte werden auf die ursprünglichen zurückgesetzt
process.env.NODE_ENV === 'development'
import.meta.env.NODE_ENV === 'development'
```

Offizielle Dokumentation: [https://vitest.dev/api/vi.html#vi-unstuballenvs](https://vitest.dev/api/vi.html#vi-unstuballenvs) 

* **`vi.stubGlobal(name, value)`** 🌐
    * Ändert globale Variablen (in `globalThis`/`global`, `window`/`top`/`self`/`parent` in Browser-Umgebungen).

* **`vi.unstubAllGlobals()`** 🌐
    * Stellt alle mit `vi.stubGlobal` geänderten globalen Variablen auf ihre ursprünglichen Werte zurück.

---

### ⏱️ Fake Timers

* **`vi.advanceTimersByTime(ms)`** ⏩
    * Führt Timer aus, bis `ms` Millisekunden vergangen sind oder die Timer-Queue leer ist.

* **`vi.advanceTimersByTimeAsync(ms)`** ⏩
    * Wie `vi.advanceTimersByTime`, aber für asynchrone Timer (Promises).

* **`vi.advanceTimersToNextTimer()`**  ➡️
    * Führt den nächsten verfügbaren Timer aus. Verkettbar für manuelle Timer-Steuerung.

* **`vi.advanceTimersToNextTimerAsync()`** ➡️
    * Wie `vi.advanceTimersToNextTimer`, aber für asynchrone Timer.

* **`vi.advanceTimersToNextFrame()`** 🖼️
    * Führt Timer für `requestAnimationFrame` Callbacks aus.

* **`vi.getTimerCount()`** 🔢
    * Gibt die Anzahl wartender Timer zurück.

* **`vi.clearAllTimers()`** 🗑️
    * Entfernt alle geplanten Timer.

* **`vi.getMockedSystemTime()`** 🕰️
    * Gibt die gemockte aktuelle Zeit zurück (Datum). `null` wenn Zeit nicht gemockt.

* **`vi.getRealSystemTime()`** ⏱️
    * Gibt die echte Zeit in Millisekunden zurück (auch wenn Fake Timers aktiv).

* **`vi.runAllTicks()`** ⚡
    * Führt alle Microtasks aus, die mit `process.nextTick` geplant wurden.

* **`vi.runAllTimers()`** 🚀
    * Führt **alle** Timer aus, bis Timer-Queue leer ist (auch neu erstellte Timer).

* **`vi.runAllTimersAsync()`** 🚀
    * Wie `vi.runAllTimers`, aber für asynchrone Timer.

* **`vi.runOnlyPendingTimers()`** ⏳
    * Führt nur Timer aus, die **nach** `vi.useFakeTimers` erstellt wurden (keine neuen Timer währenddessen).

* **`vi.runOnlyPendingTimersAsync()`** ⏳
    * Wie `vi.runOnlyPendingTimers`, aber für asynchrone Timer.

* **`vi.setSystemTime(date)`** 📅
    * Setzt die gemockte Systemzeit (Datum). Beeinflusst `Date.*` Aufrufe.

* **`vi.useFakeTimers(config?)`** 🕰️
    * Aktiviert Fake Timers. Mockt `setTimeout`, `setInterval`, `Date`, etc.
    * `config?`: Optionen für Fake Timers (z.B. `toFake: ['nextTick', 'queueMicrotask']`).

* **`vi.isFakeTimers()`** ✅
    * Gibt `true` zurück, wenn Fake Timers aktiviert sind.

* **`vi.useRealTimers()`** 🕰️
    * Deaktiviert Fake Timers, stellt originale Timer-Implementierungen wieder her.

---

### ✨ Miscellaneous

* **`vi.waitFor<T>(callback, options?)`** ⏳
    * Wartet, bis `callback` erfolgreich ausgeführt wird (kein Fehler, Promise resolved).
    * `options?`: `{ timeout?: number, interval?: number }` oder `number` (für `timeout`).
    * Ruft automatisch `vi.advanceTimersByTime(interval)` auf, wenn `vi.useFakeTimers` aktiv.

    ```typescript
    await vi.waitFor(() => {
      expect(server.isReady).toBe(true); // Throw Error wenn nicht ready
    }, { timeout: 500 });
    ```

* **`vi.waitUntil<T>(callback, options?)`** ⏳
    * Ähnlich `vi.waitFor`, aber bricht sofort bei Fehler im `callback` ab.
    * Wartet auf einen Truthy-Rückgabewert von `callback`.

    ```typescript
    const element = await vi.waitUntil(() => document.querySelector('.element'));
    expect(element).toBeTruthy();
    ```

* **`vi.hoisted<T>(factory)`** ⬆️
    * Ermöglicht Code-Ausführung **vor** statischen Imports.
    * Konvertiert statische Imports in dynamische (mit Live-Bindings).
    * Rückgabewert von `factory` kann in `vi.mock` Factories genutzt werden.
    * **Achtung:** Kein Zugriff auf importierte Variablen innerhalb `vi.hoisted` (da noch nicht definiert).

    ```typescript
    const mockedFn = vi.hoisted(() => vi.fn());

    vi.mock('./modul.js', () => ({
      exportedFn: mockedFn
    }));
    ```

* **`vi.setConfig(config)`** ⚙️
    * Aktualisiert die Vitest Konfiguration **für die aktuelle Testdatei**.
    * Unterstützt Optionen wie `allowOnly`, `testTimeout`, `fakeTimers`, etc.

* **`vi.resetConfig()`** ⚙️
    * Setzt die Konfiguration auf den ursprünglichen Zustand zurück (falls `vi.setConfig` verwendet).
