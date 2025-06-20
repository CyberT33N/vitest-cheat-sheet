# Mock Factory External Strategies - Prioritätsliste & Best Practices

## 🎯 **Zusammenfassung & Prioritäten**

**Für Runtime-Mocking ist `vi.mock()` mit async Callback die moderne, zukunftsorientierte Lösung** - sie bietet bessere TypeScript-Integration, automatisches Cleanup und konsistentere API im Vergleich zu `vi.doMock()`. **Für Hoisted-Mocking bleibt `vi.mocked()` der Gold-Standard** mit optimaler Performance und einfachster Syntax.

---

## 🚀 **Runtime Mock Strategies** (Zur Laufzeit)

> **⚠️ WICHTIG:** Runtime-Mocks **MÜSSEN** nur in **Edge-Cases** verwendet werden, wo andere Dependencies ein gemocktes Modul aufrufen und es bei der Hoisted-Variante zu früh gemockt wird. **PRIORISIERE IMMER ZUERST die Hoisted-Mock-Strategien** - nur wenn Hoisting-Konflikte auftreten, wechsle zur Runtime-Variante.

### 🥇 **#1 EMPFOHLEN: `vi.mock()` mit Async Callback**
- **Datei:** [`custom __mocks__/runtime/viMock-callback.md`](./custom%20__mocks__/runtime/viMock-callback.md)
- **Warum besser:** Moderne Vitest-Best-Practice, automatisches Cleanup, bessere TypeScript-Integration, konsistentere API
- **Ideal für:** NPM Package FS-Konflikte, komplexe Mock-Factories, dynamische Mocks

### 🥈 **#2 ALTERNATIVE: `vi.doMock()`**
- **Datei:** [`internal __mocks__/runtime/doMock.md`](./internal%20__mocks__/runtime/doMock.md)
- **Warum zweitrangig:** Zwei-Schritt-Prozess, manuelles Cleanup erforderlich, mehr Boilerplate
- **Ideal für:** Legacy-Code, spezielle Edge-Cases, wenn Callback-Approach nicht funktioniert

---

## ⚡ **Hoisted Mock Strategies** (Kompilierzeit)

### 🏆 **BEST PRACTICE: `vi.mocked()`**
- **Datei:** [`internal __mocks__/hoisted/vi.mocked.md`](./internal%20__mocks__/hoisted/vi.mocked.md)
- **Warum optimal:** Beste Performance, einfachste Syntax, automatisches Hoisting, perfekte TypeScript-Typisierung
- **Ideal für:** Standard-Mocking, maximale Performance, einfache Test-Setups

---

## 📋 **Quick Decision Guide**

| Szenario | Empfohlene Strategie | Begründung |
|----------|---------------------|------------|
| **NPM Package FS-Konflikte** | `vi.mock()` + Callback | Vermeidet Hoisting-Probleme |
| **Dynamische Mock-Konfiguration** | `vi.mock()` + Callback | Flexibelste Runtime-Kontrolle |
| **Standard Unit Tests** | `vi.mocked()` | Beste Performance + einfachste Syntax |
| **Legacy Code Migration** | `vi.doMock()` | Schrittweise Modernisierung möglich |
| **Komplexe Mock Factories** | `vi.mock()` + Callback | Modularste und wartbarste Lösung |
