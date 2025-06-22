# Partielle Mocks mit Custom `__mocks__`-Ordnern

Dieser Abschnitt behandelt fortgeschrittene Architekturen für das partielle Mocking von Modulen unter Verwendung eines benutzerdefinierten `__mocks__`-Ordners. Anders als der root-level `__mocks__`-Ordner, der von Vitest automatisch erkannt wird, dient dieser Ordner zur Organisation von wiederverwendbaren, zentralen Mock-Dateien, die explizit in den Tests aktiviert werden.

Diese Techniken sind besonders mächtig, um komplexe Abhängigkeiten zu managen und Hoisting-Probleme zu lösen.

---

## 📁 Verzeichnisstruktur & Beispiele

```
.
├── hoisted/
│   ├── multiple-test-files.md
│   └── single-test-file.md
├── runtime/
│   └── viMock-callback.md
└── index.md
```

---

## 🏆 Priorisierte Architekturen & Empfehlungen

Es gibt zwei primäre Ansätze, die hier vorgestellt werden. Die Wahl hängt von der Komplexität Ihres Setups ab.

### 🥇 1. Priorität: `vi.hoisted()`-Architektur (Bevorzugt)
Diese Methode ist die **robusteste und empfohlene Lösung** für die meisten Anwendungsfälle.

-   **Wie es funktioniert:** Eine zentrale Mock-Datei nutzt `vi.hoisted()`, um Mock-Implementierungen zu definieren, *bevor* `vi.mock()` aufgerufen wird. Ein kritischer, leerer Import (`import 'pfad/zur/mock-datei'`) in der Testdatei aktiviert die Mocks.
-   **Vorteile:** Löst proaktiv die meisten Hoisting-Probleme, ist sehr sauber strukturiert und gut wartbar.
-   **Wann verwenden:** **Immer als erste Wahl.** Dies sollte Ihr Standardansatz sein.

### 🥈 2. Priorität: Runtime-Mock mit `vi.mock()`-Callback (Fallback)
Diese Methode ist eine mächtige **Alternative**, wenn die Hoisted-Variante an ihre Grenzen stößt.

-   **Wie es funktioniert:** `vi.mock()` wird mit einer asynchronen Callback-Funktion verwendet, die den Mock zur Laufzeit dynamisch importiert (`await import(...)`).
-   **Vorteile:** Funktioniert auch in komplexen Szenarien, in denen das zu mockende Modul bereits an anderer Stelle importiert wurde, bevor die Test-Mocks greifen konnten.
-   **Wann verwenden:** Nur dann, wenn die `vi.hoisted()`-Methode nachweislich fehlschlägt. Dies ist ein Indiz für komplexe Import-Reihenfolgen im Projekt.

---

## 📋 Inhaltsverzeichnis der Varianten

| Architektur | Beschreibung | Anwendungsfall |
| :--- | :--- | :--- |
| [**Hoisted Mock (Multi-File)**](./hoisted/multiple-test-files.md) | **(Bevorzugt)** Zeigt die `vi.hoisted()`-Architektur für Tests, die über mehrere Dateien verteilt sind. | Ideal für komplexe Services, deren Tests modular aufgeteilt sind. |
| [**Hoisted Mock (Single-File)**](./hoisted/single-test-file.md) | **(Bevorzugt)** Ein vereinfachtes Beispiel der `vi.hoisted()`-Architektur für eine einzelne, übersichtliche Testdatei. | Der Standard für die meisten Unit-Tests, die einen einzelnen Service testen. |
| [**Runtime Mock (Callback)**](./runtime/viMock-callback.md) | **(Fallback)** Nutzt `vi.mock()` mit einem asynchronen Callback, um Hoisting-Probleme zur Laufzeit zu umgehen. | Als Problemlöser, wenn die Hoisted-Architektur aufgrund früherer Modul-Imports fehlschlägt. |
