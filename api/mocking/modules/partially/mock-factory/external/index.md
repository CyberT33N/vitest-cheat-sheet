# Architektonischer Leitfaden: Mocking-Strategien in Vitest

Dieses Dokument dient als Referenz für die Auswahl der optimalen Mocking-Strategie für externe NPM-Module in einem Vitest-basierten Test-Setup. Die Wahl der Strategie hängt maßgeblich vom Ausführungszeitpunkt des Mocks (Kompilierzeit vs. Laufzeit) und der Komplexität des zu mockenden Moduls ab.

## Priorisierte Implementierungs-Reihenfolge

> **⚠️ WICHTIG:** Die folgende Reihenfolge ist eine **strategische Entscheidungsmatrix** basierend auf der **Modul-Architektur**. Die Wahl der Strategie hängt **DIREKT** von der **Komplexität und Struktur** des zu mockenden Moduls ab, nicht nur von Problemen mit der ersten Strategie.

### **Strategie 1: Hoisted Mocking - SIMPLE STANDARDMODULE**
**Status:** ⭐ **FÜR EINFACHE, ZUSTANDSLOSE MODULE**
- **Zielgruppe:** **Funktionsbasierte Module** ohne komplexe Instanziierung
- **Perfekt für:** `fs`, `axios`, `lodash`, `path`, `crypto`, `os`
- **Mechanismus:** `__mocks__/[module].ts` + `vi.mock('[module]')` in Setup-Dateien
- **Charakteristika:** Module mit **direkten Funktionsexporten** oder **einfachen Default-Exports**
- **Erkennungsmerkmale:**
  ```typescript
  // ✅ Geeignet für Strategie 1:
  import fs from 'fs'              // Einfacher Namespace
  import axios from 'axios'        // Direkter Default-Export
  import _ from 'lodash'           // Funktionssammlung
  fs.readFileSync(...)             // Direkte Funktionsaufrufe
  axios.get(...)                   // Statische Methodenaufrufe
  ```

### **Strategie 2: Runtime Mocking - KOMPLEXE SDK-KLASSEN**
**Status:** 🏗️ **DIREKT FÜR KLASSENBASIERTE SDKs**
- **Zielgruppe:** **Instanzbasierte SDKs** mit komplexer Objekthierarchie
- **Designed für:** `@pinecone-database/pinecone`, `@aws-sdk/*`, `firebase`, `mongodb`, `redis`
- **Mechanismus:** Mock-Factory-Pattern + `vi.mock('[module]', async () => { ... })`
- **Charakteristika:** Module mit **Konstruktor-basierten APIs** und **verketteten Instanzen**
- **Erkennungsmerkmale:**
  ```typescript
  // ✅ Benötigt DIREKT Strategie 2:
  import { Pinecone } from '@pinecone-database/pinecone'
  const pinecone = new Pinecone({ apiKey: '...' })    // Klassen-Instanziierung
  const index = pinecone.index('my-index')            // Verkettete Objekterstellung
  await index.namespace('ns').upsert([...])          // Verschachtelte Methodenketten
  
  // ✅ Weitere Beispiele:
  const client = new MongoClient(uri)                 // Constructor-Pattern
  const db = client.db('mydb')                        // Instanz-Methoden
  const collection = db.collection('users')          // Verkettete Objekterstellung
  ```
- **WICHTIG:** Diese Module **KÖNNEN NICHT** mit Strategie 1 gemockt werden, da die **Mock-Factory-Logik** für **Klasseninstanzen** zwingend erforderlich ist

### **Strategie 3: Kritische Hoisting-Konflikte - LAST RESORT**
**Status:** 🚨 **NUR bei schwerwiegenden Architektur-Konflikten**
- **Anwendungsfall:** Wenn **Strategie 1 oder 2** durch **Lade-Reihenfolge-Konflikte** fehlschlagen
- **Mechanismus:** `vi.doMock('[module]')` in `beforeAll()` + **dynamische Imports**
- **Edge-Cases:** **Bootstrap-Korruption**, **zirkuläre Dependencies**, **Electron-Prozess-Konflikte**

### **Entscheidungsmatrix: Welche Strategie für welches Modul?**

| Modul-Typ | API-Stil | Strategie | Begründung |
|-----------|----------|-----------|------------|
| **File System** | `fs.readFileSync()` | **Strategie 1** | Einfache Funktions-APIs |
| **HTTP Client** | `axios.get()` | **Strategie 1** | Direkter Default-Export |
| **Utilities** | `_.map()`, `uuid()` | **Strategie 1** | Zustandslose Funktionen |
| **Database SDK** | `new Client()` → `client.db()` | **Strategie 2** | Instanz-verkettete APIs |
| **Vector DB** | `new Pinecone()` → `index()` | **Strategie 2** | Komplexe Objekthierarchie |
| **Cloud SDK** | `new S3()` → `bucket()` | **Strategie 2** | Service-Instanz-Pattern |
| **Auth SDK** | `new Auth()` → `user()` | **Strategie 2** | Zustandsbehaftete Klassen |

### **Schnell-Identifikation der richtigen Strategie:**

#### **→ Strategie 1 Indikatoren:**
```typescript
// ✅ DIREKT erkennbar - einfache Module:
import fs from 'fs'
import axios from 'axios'
import path from 'path'

// Nutzung ohne `new` Keyword:
fs.existsSync('/path/to/file')
axios.get('https://api.example.com')
path.join('/', 'users', 'profile')
```

#### **→ Strategie 2 Indikatoren:**
```typescript
// ✅ DIREKT erkennbar - komplexe SDK-Klassen:
import { Pinecone, Index } from '@pinecone-database/pinecone'
import { MongoClient } from 'mongodb'
import { S3Client } from '@aws-sdk/client-s3'

// Nutzung MIT `new` Keyword und Instanz-Ketten:
const client = new Pinecone({ apiKey: 'key' })        // Constructor
const index = client.index('my-index')                 // Instance method
const namespace = index.namespace('production')        // Chained instances

const mongo = new MongoClient(uri)                      // Constructor
const database = mongo.db('myapp')                     // Instance method
const users = database.collection('users')             // Chained instances
```

---

## ⚡ Strategie 1: Hoisted Mocking (Kompilierzeit)

Dies ist der "Gold-Standard" für das Mocking in Vitest. Er nutzt die Fähigkeit von Vitest, Modul-Imports zur Kompilierzeit zu erkennen und durch Mocks zu ersetzen, bevor der eigentliche Testcode ausgeführt wird. **Diese Strategie MUSS immer zuerst versucht werden.**

### 🥇 **#1 BEST PRACTICE (Standard): Automatisches Mocking via `__mocks__`**
- **Referenz-Datei:** [`internal __mocks__/hoisted/vi.mocked.md`](./internal%20__mocks__/hoisted/vi.mocked.md)
- **Warum optimal:** Beste Performance, einfachste Syntax, automatisches Hoisting, perfekte TypeScript-Typisierung in Kombination mit `vi.mocked()`.
- **Ideal für:** Standard-Mocking einfacher, zustandsloser Bibliotheken (`axios`, `lodash`).

#### **Konzept & Anwendungsfall: Das "Axios-Modell"**
Ein `__mocks__`-Verzeichnis im Projekt-Root enthält eine Mock-Implementierung (z.B., `__mocks__/axios.ts`). Ein einfacher Aufruf von `vi.mock('axios')` aktiviert diesen Mock global und implizit für den gesamten Testlauf. Dies ist perfekt für Bibliotheken, bei denen einfache Funktionsüberschreibungen ausreichen.

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