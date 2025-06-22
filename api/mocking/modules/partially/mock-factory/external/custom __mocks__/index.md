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

### 🥇 1. Priorität: Runtime-Mock mit `vi.mock()`-Callback (Bevorzugt)
Diese Methode ist die **robusteste und empfohlene Lösung** für komplexe Anwendungsfälle.

-   **Wie es funktioniert:** `vi.mock()` wird mit einer asynchronen Callback-Funktion verwendet, die den Mock zur Laufzeit dynamisch importiert (`await import(...)`).
-   **Vorteile:** Funktioniert auch in komplexen Szenarien, in denen das zu mockende Modul bereits an anderer Stelle importiert wurde, bevor die Test-Mocks greifen konnten. Sehr mächtig für Enterprise-Setups.
-   **Wann verwenden:** **Als erste Wahl für komplexe Module.** Dies sollte Ihr Standardansatz für fortgeschrittene Mocking-Szenarien sein.

### 🥈 2. Priorität: `vi.hoisted()`-Architektur (Alternative)
Diese Methode ist eine **saubere Alternative** für einfachere Anwendungsfälle.

-   **Wie es funktioniert:** Eine zentrale Mock-Datei nutzt `vi.hoisted()`, um Mock-Implementierungen zu definieren, *bevor* `vi.mock()` aufgerufen wird. Ein kritischer, leerer Import (`import 'pfad/zur/mock-datei'`) in der Testdatei aktiviert die Mocks.
-   **Vorteile:** Löst proaktiv die meisten Hoisting-Probleme, ist sehr sauber strukturiert und gut wartbar.
-   **Wann verwenden:** Wenn die Runtime-Callback-Variante zu komplex erscheint oder für einfachere Module ausreichend ist.

---

## 📋 Inhaltsverzeichnis der Varianten

| Architektur | Beschreibung | Anwendungsfall |
| :--- | :--- | :--- |
| [**Runtime Mock (Callback)**](./runtime/viMock-callback.md) | **(Bevorzugt)** Nutzt `vi.mock()` mit einem asynchronen Callback für robuste Mocking-Strategien. | Erste Wahl für komplexe Module und Enterprise-Setups mit komplexen Import-Abhängigkeiten. |
| [**Hoisted Mock (Multi-File)**](./hoisted/multiple-test-files.md) | **(Alternative)** Zeigt die `vi.hoisted()`-Architektur für Tests, die über mehrere Dateien verteilt sind. | Gute Option für komplexe Services, wenn Runtime-Mocks zu aufwändig erscheinen. |
| [**Hoisted Mock (Single-File)**](./hoisted/single-test-file.md) | **(Alternative)** Ein vereinfachtes Beispiel der `vi.hoisted()`-Architektur für eine einzelne, übersichtliche Testdatei. | Solide Basis für einfachere Unit-Tests, die einen einzelnen Service testen. |
