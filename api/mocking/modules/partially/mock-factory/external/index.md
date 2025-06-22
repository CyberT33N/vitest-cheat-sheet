# Architektonischer Leitfaden: Mocking-Strategien in Vitest

Dieses Dokument dient als Referenz für die Auswahl der optimalen Mocking-Strategie für externe NPM-Module in einem Vitest-basierten Test-Setup. Die Wahl der Strategie hängt maßgeblich vom Ausführungszeitpunkt des Mocks (Kompilierzeit vs. Laufzeit) und der Komplexität des zu mockenden Moduls ab.

## Prioritätsliste & Best Practices

> **⚠️ WICHTIG:** Bevorzugen Sie **IMMER** Hoisted Mock Strategies. Runtime-Mocks sind eine leistungsstarke, aber sekundäre Option, die nur dann eingesetzt werden sollte, wenn es zu unauflösbaren Konflikten durch das Hoisting kommt (z.B. wenn eine andere Abhängigkeit das Originalmodul benötigt, bevor der Mock initialisiert werden kann).

1.  🏆 **Hoisted (Kompilierzeit):** Die schnellste, einfachste und bevorzugte Methode für Standardfälle.
2.  🥇 **Runtime (Laufzeit):** Die flexible Lösung für komplexe SDKs und Hoisting-Konflikte.

---

## ⚡ Strategie 1: Hoisted Mocking (Kompilierzeit)

Dies ist der "Gold-Standard" für das Mocking in Vitest. Er nutzt die Fähigkeit von Vitest, Modul-Imports zur Kompilierzeit zu erkennen und durch Mocks zu ersetzen, bevor der eigentliche Testcode ausgeführt wird.

### 🏆 **BEST PRACTICE: Automatisches Mocking via `__mocks__`-Verzeichnis**
- **Referenz-Datei:** [`internal __mocks__/hoisted/vi.mocked.md`](./internal%20__mocks__/hoisted/vi.mocked.md)
- **Warum optimal:** Beste Performance, einfachste Syntax, automatisches Hoisting, perfekte TypeScript-Typisierung in Kombination mit `vi.mocked()`.
- **Ideal für:** Standard-Mocking, maximale Performance, einfache Test-Setups.

#### **Anwendungsfall: Das "Axios-Modell"**

Dieser Ansatz ist ideal für einfache, weitgehend zustandslose Bibliotheken wie `axios`.

#### **Konzept**

1.  **Ort:** Ein `__mocks__`-Verzeichnis wird im Projekt-Root angelegt.
2.  **Datei:** Innerhalb dieses Ordners wird eine Datei erstellt, die exakt so heißt wie das zu mockende NPM-Package (z.B. `__mocks__/axios.ts`).
3.  **Aktivierung:** Ein einfacher Aufruf von `vi.mock('axios')` in einer Test-Setup-Datei oder direkt im Test-File genügt. Vitest findet und verwendet automatisch die Datei aus dem `__mocks__`-Verzeichnis.

#### **Architektonische Merkmale**

*   **Global & Implizit:** Der Mock wird systemweit für den Testlauf aktiviert. Es ist eine Form von "magischem" Hintergrundverhalten.
*   **Zustandslos-Fokus:** Ideal, um einzelne, exportierte Funktionen (`get`, `post`) zu überschreiben.
*   **Wenig Boilerplate:** Tests benötigen keinen expliziten Import oder Setup für den Mock, was den Testcode schlank hält.

#### **Entscheidungskriterien**

**Verwenden Sie diese Strategie, wenn ALLE folgenden Kriterien zutreffen:**

*   **✅ Einfache API:** Das Modul exportiert hauptsächlich Funktionen oder ein Singleton-Objekt (z.B. `axios.get`).
*   **✅ Zustandslos:** Die Funktionen haben keine internen Zustände, die sich über Aufrufe hinweg ändern müssen.
*   **✅ Globale Mocks ausreichend:** Ein einziges, globales Mock-Verhalten ist für die Mehrheit der Tests ausreichend.

---

## 🚀 Strategie 2: Runtime Mocking (Zur Laufzeit)

Dieser Ansatz wird verwendet, wenn Hoisting zu Problemen führt oder wenn eine komplexe, instanzbasierte Bibliothek gemockt werden muss, die mehr Kontrolle erfordert als ein globaler Mock bieten kann.

### 🥇 **#1 EMPFOHLEN: `vi.mock()` mit Async Callback & Mock-Factory**
- **Referenz-Datei:** [`custom __mocks__/runtime/viMock-callback.md`](./custom%20__mocks__/runtime/viMock-callback.md)
- **Warum besser:** Moderne Vitest-Best-Practice, umgeht Hoisting-Konflikte, automatisches Cleanup, exzellente TypeScript-Integration für komplexe Szenarien.
- **Ideal für:** Konflikte mit anderen Dependencies (z.B. `fs`), komplexe Mock-Factories, dynamische Mocks.

### 🥈 **#2 ALTERNATIVE: `vi.doMock()`**
- **Datei:** [`internal __mocks__/runtime/doMock.md`](./internal%20__mocks__/runtime/doMock.md)
- **Warum zweitrangig:** Zwei-Schritt-Prozess, manuelles Cleanup erforderlich, mehr Boilerplate
- **Ideal für:** Legacy-Code, spezielle Edge-Cases, wenn Callback-Approach nicht funktioniert

#### **Anwendungsfall: Das "Pinecone-Modell"**

Dieser Ansatz ist eine bewusste, manuelle und kontrollierte Methode, die für komplexe, instanzbasierte SDKs wie `@pinecone-database/pinecone` entwickelt wurde.

#### **Konzept**

1.  **Die Herausforderung:** Komplexe SDKs erfordern oft eine Kette von Instanziierungen (`new Pinecone() -> index() -> namespace() -> upsert()`). Ein einfacher Mock scheitert hier.
2.  **Die Lösung (Mock-Factory):**
    *   **Ort:** Eine Factory-Klasse (z.B. `PineconeMockFactory`) wird in einem dedizierten Test-Verzeichnis angelegt (z.B. `test/mocks/`).
    *   **Design:** Die Factory ist dafür verantwortlich, vollständig gemockte Instanzen (`mockPineconeInstance`, `mockIndexInstance`) zu erstellen und bereitzuhalten.
    *   **Aktivierung:** Der Mock wird explizit und pro Test-Suite mit `vi.mock('@pinecone-database/pinecone', ...)` und einem **asynchronen Callback**, der die Factory importiert, aktiviert. Dies gibt volle Kontrolle und umgeht Hoisting-Probleme.

#### **Architektonische Merkmale**

*   **Explizit & Kontrolliert:** Der Mock wird sichtbar im Testcode importiert und konfiguriert.
*   **Zustandsbehaftet & Isoliert:** Perfekt für instanzbasierte SDKs. Jeder Test kann durch `factory.resetAllMocks()` mit einem sauberen, definierten Zustand beginnen.
*   **Flexibilität & Wartbarkeit:** Das Verhalten kann pro Testfall (`it`-Block) präzise angepasst werden.

#### **Entscheidungskriterien**

**Verwenden Sie diese Strategie, wenn EINES der folgenden Kriterien zutrifft:**

*   **✅ Instanzbasiertes SDK:** Der Code verwendet `new SDK()` zur Initialisierung.
*   **✅ Gekettete Methodenaufrufe:** Der Zugriff auf die Zielmethode erfordert mehrere vorherige Aufrufe, die jeweils neue Objekte zurückgeben.
*   **✅ Hoisting-Konflikte:** Die Hoisted-Strategie führt zu Fehlern, weil andere Module das Original-Package benötigen.
*   **✅ Detaillierte Test-Kontrolle:** Sie benötigen für verschiedene Tests unterschiedliche Rückgabewerte oder Fehlerszenarien derselben Methode.

---

### Direkter Vergleich & Fazit

| Merkmal | Strategie 1 (Hoisted Mocking) | Strategie 2 (Runtime Mocking mit Factory) |
| :--- | :--- | :--- |
| **Aktivierung** | Implizit & zur Kompilierzeit | Explizit & zur Laufzeit |
| **Kontrolle** | Global, geringe Flexibilität pro Test | Pro Test, maximale Flexibilität |
| **Isolation** | Geringer, da globaler Zustand | Sehr hoch, da Mocks pro Test zurückgesetzt werden können |
| **Ideal für...** | Einfache, zustandslose Funktionsbibliotheken | Komplexe, instanzbasierte SDKs & Hoisting-Konflikte |
| **Gefahren** | Hoisting-Konflikte, schwer zu debuggende globale Zustände | Geringfügig mehr initialer Einrichtungsaufwand |

**Schlussfolgerung für AI-Agents:**
Die Wahl der Mocking-Strategie ist eine architektonische Entscheidung.

*   Beginnen Sie immer mit der **Hoisted-Strategie** für Einfachheit und Performance.
*   Für das Testen von Interaktionen mit einem komplexen, instanzbasierten Dienst wie **Pinecone** oder bei Hoisting-Konflikten ist der Wechsel zur **Runtime-Strategie mit einer expliziten Mock-Factory** nicht nur sinnvoll, sondern **zwingend erforderlich**, um robuste und wartbare Tests zu gewährleisten.