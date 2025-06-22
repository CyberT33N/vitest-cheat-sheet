# Architektonischer Leitfaden: Mocking-Strategien in Vitest

Dieses Dokument dient als Referenz für die Auswahl der optimalen Mocking-Strategie für externe NPM-Module in einem Vitest-basierten Test-Setup. Die Wahl der Strategie hängt maßgeblich vom Ausführungszeitpunkt des Mocks (Kompilierzeit vs. Laufzeit) und der Komplexität des zu mockenden Moduls ab.

## Priorisierte Implementierungs-Reihenfolge

> **⚠️ WICHTIG:** Die folgende Reihenfolge ist eine **strikte Handlungsanweisung**. Beginnen Sie immer mit der ersten Option und gehen Sie nur dann zur nächsten über, wenn die vorherige für den Anwendungsfall ungeeignet ist.
>
> 1.  **(Standard) Hoisted via `__mocks__`:** Immer die erste Wahl für einfache Module.
> 2.  **(Fallback) Hoisted via Zentrale Mock-Datei:** Der zweite Schritt für komplexe SDKs, die noch gehoisted werden können.
> 3.  **(Edge Case) Runtime:** Die letzte Option, wenn Hoisting aufgrund von Lade-Reihenfolge-Konflikten fehlschlägt.

---

## ⚡ Strategie 1: Hoisted Mocking (Kompilierzeit)

Dies ist der "Gold-Standard" für das Mocking in Vitest. Er nutzt die Fähigkeit von Vitest, Modul-Imports zur Kompilierzeit zu erkennen und durch Mocks zu ersetzen, bevor der eigentliche Testcode ausgeführt wird. **Diese Strategie MUSS immer zuerst versucht werden.**

### 🥇 **#1 BEST PRACTICE (Standard): Automatisches Mocking via `__mocks__`**
- **Referenz-Datei:** [`internal __mocks__/hoisted/vi.mocked.md`](./internal%20__mocks__/hoisted/vi.mocked.md)
- **Warum optimal:** Beste Performance, einfachste Syntax, automatisches Hoisting, perfekte TypeScript-Typisierung in Kombination mit `vi.mocked()`.
- **Ideal für:** Standard-Mocking einfacher, zustandsloser Bibliotheken (`axios`, `lodash`).

#### **Konzept & Anwendungsfall: Das "Axios-Modell"**
Ein `__mocks__`-Verzeichnis im Projekt-Root enthält eine Mock-Implementierung (z.B., `__mocks__/axios.ts`). Ein einfacher Aufruf von `vi.mock('axios')` aktiviert diesen Mock global und implizit für den gesamten Testlauf. Dies ist perfekt für Bibliotheken, bei denen einfache Funktionsüberschreibungen ausreichen.

### 🥈 **#2 FALLBACK: Zentrale Mock-Datei mit `vi.hoisted()`**
- **Referenz-Datei:** [`custom __mocks__/hoisted/single-test-file.md`](./custom%20__mocks__/hoisted/single-test-file.md)
- **Warum ein guter Fallback:** Diese Methode ist die **Hoisted-Variante des "Pinecone-Modells"**. Sie bietet die gleiche saubere Trennung von Mock- und Test-Logik wie die Runtime-Factory, nutzt aber die Vorteile des Hoistings für eine bessere Performance, solange keine kritischen Ladekonflikte vorliegen.
- **Ideal für:** **Komplexe, instanzbasierte SDK-Architekturen** (ähnlich dem Pinecone-Beispiel), bei denen die automatische Mock-Methode nicht ausreicht, aber ein Runtime-Mock (Strategie 2) noch nicht zwingend notwendig ist.

---

## 🚀 Strategie 2: Runtime Mocking (Zur Laufzeit)

Dieser Ansatz wird verwendet, **wenn die beiden Hoisted-Methoden aus Strategie 1 fehlschlagen**. Dies geschieht typischerweise, wenn Hoisting zu Problemen führt oder wenn eine komplexe, instanzbasierte Bibliothek gemockt werden muss, die mehr Kontrolle erfordert.

### 🥇 **#1 EMPFOHLEN: `vi.mock()` mit Async Callback & Mock-Factory**
- **Referenz-Datei:** [`custom __mocks__/runtime/viMock-callback.md`](./custom%20__mocks__/runtime/viMock-callback.md)
- **Warum besser:** Moderne Vitest-Best-Practice, die Hoisting-Konflikte umgeht, automatisches Cleanup bietet und eine exzellente TypeScript-Integration für komplexe Szenarien sicherstellt.
- **Ideal für:** Komplexe, instanzbasierte SDKs (`@pinecone-database/pinecone`, `aws-sdk`) und die meisten Hoisting-Konflikte.

#### **Konzept & Anwendungsfall: Das "Pinecone-Modell"**
Für SDKs, die eine Instanziierungskette erfordern (`new SDK().feature().action()`), wird eine Mock-Factory in den Test-Dateien erstellt. Diese Factory liefert vollständig präparierte Mock-Instanzen. Die Aktivierung erfolgt explizit über `vi.mock('module', async () => { ... })`, was eine feingranulare Kontrolle pro Test-Suite ermöglicht und die Testlogik sauber vom Mock-Setup trennt.

### 🥈 **#2 FALLBACK FÜR EDGE-CASES: `vi.doMock()`**
- **Referenz-Datei:** [`internal __mocks__/runtime/doMock.md`](./internal%20__mocks__/runtime/doMock.md)
- **Warum nur für Edge-Cases:** Erfordert manuelle, dynamische Imports und ist weniger elegant als der Callback-Ansatz.
- **Ideal für:** **Kritische Hoisting-Konflikte**, bei denen eine Dependency (z.B. ein Test-Setup-Skript) eine andere Dependency (z.B. `fs`) lädt, die *vor* dem `vi.mock`-Callback ausgeführt werden muss. Wenn `vi.mock` also selbst für Runtime zu früh ist, ist `vi.doMock` die letzte Instanz.

#### **Konzept & Anwendungsfall: Kritische Lade-Reihenfolge**
`vi.doMock()` wird innerhalb eines Test-Hooks (z.B. `beforeAll`) aufgerufen. Dies garantiert, dass der Mock erst zur Laufzeit initialisiert wird. Der **entscheidende Nachteil**: Alle Module, die von dem gemockten Modul abhängen, **müssen zwingend *nach* dem `vi.doMock()`-Aufruf dynamisch mit `await import(...)` geladen werden.** Top-Level-Imports für diese Dateien sind hier nicht mehr möglich.

---

### Fazit für die Strategiewahl

| Merkmal | Strategie 1 (Hoisted) | Strategie 2 (Runtime / Factory) | Strategie 2 (Runtime / `doMock`) |
| :--- | :--- | :--- | :--- |
| **Aktivierung** | Implizit, Kompilierzeit | Explizit, Laufzeit (via Callback) | Explizit, Laufzeit (im Test-Hook) |
| **Ideal für** | Einfache, zustandslose Module | Komplexe, instanzbasierte SDKs | Kritische Hoisting-Konflikte |
| **Gefahren** | Hoisting-Konflikte | - | Manuelle dynamische Imports nötig |

**Regel für autonome Agents:**
1.  **Versuche immer Strategie 1 (Hoisted).**
2.  Bei Komplexität oder Hoisting-Konflikten, **wechsle zu Strategie 2 (`vi.mock` mit Callback).**
3.  Nur wenn selbst der `vi.mock`-Callback noch zu Konflikten führt, **nutze als letzte Option Strategie 2 mit `vi.doMock()`** und beachte die Regel der dynamischen Imports.