# Mocking Modules



## vitest/mocker
- https://github.com/vitest-dev/vitest/blob/main/packages/mocker/EXPORTS.md



<br><br>

---

<br><br>


# Mocking von Modulen / NPM-Packages in Vitest

Dieses Verzeichnis enthält eine umfassende Sammlung von Beispielen und Anleitungen für das Mocking von Modulen und NPM-Packages in Vitest. Die Inhalte sind nach verschiedenen Mocking-Strategien organisiert.

## 📋 Inhaltsverzeichnis

### 🔄 Auto Mocking
Automatisches Mocking von Modulen ohne explizite Konfiguration.

| Datei | Thematik | Beschreibung |
|-------|----------|--------------|
| [`auto/default.md`](./auto/default.md) | Standard Auto-Mocking | Grundlagen des automatischen Mockings in Vitest |

---

### 🎯 Manually Mocking
Manuelles Mocking mit vollständiger Kontrolle über Mock-Implementierungen.

#### Mock Factory (External)
| Datei | Thematik | Beschreibung |
|-------|----------|--------------|
| [`manually/mock-factory/external/__mocks__/usage-without-vi-fn-access.md`](./manually/mock-factory/external/__mocks__/usage-without-vi-fn-access.md) | Externe Mocks ohne vi.fn | Verwendung von __mocks__ Ordner ohne direkten Zugriff auf vi.fn |

#### Mock Factory (Internal)
| Datei | Thematik | Beschreibung |
|-------|----------|--------------|
| [`manually/mock-factory/internal/mockImplementation.md`](./manually/mock-factory/internal/mockImplementation.md) | mockImplementation API | Detaillierte Verwendung von mockImplementation() |
| [`manually/mock-factory/internal/vi-mock.md`](./manually/mock-factory/internal/vi-mock.md) | vi.mock() Grundlagen | Basis-Verwendung von vi.mock() für komplette Module |

#### Runtime Mocking
| Datei | Thematik | Beschreibung |
|-------|----------|--------------|
| [`manually/runtime/vi-doMock.md`](./manually/runtime/vi-doMock.md) | vi.doMock() | Dynamisches Mocking zur Laufzeit mit vi.doMock() |
| [`manually/runtime/vi-importMock.md`](./manually/runtime/vi-importMock.md) | vi.importMock() | Import und Mock in einem Schritt mit vi.importMock() |

---

### 🔀 Partially Mocking
Partielles Mocking - nur bestimmte Teile eines Moduls mocken.

#### Mock Factory (External)

##### Custom __mocks__
| Datei | Thematik | Beschreibung |
|-------|----------|--------------|
| [`partially/mock-factory/external/custom __mocks__/index.md`](./partially/mock-factory/external/custom%20__mocks__/index.md) | Custom __mocks__ Übersicht | Einführung in benutzerdefinierte __mocks__ Ordner |
| [`partially/mock-factory/external/custom __mocks__/runtime/viMock-callback.md`](./partially/mock-factory/external/custom%20__mocks__/runtime/viMock-callback.md) | vi.mock() mit Callback | Verwendung von vi.mock() mit Callback-Funktionen zur Laufzeit |

##### Externe Partial Mocks
| Datei | Thematik | Beschreibung |
|-------|----------|--------------|
| [`partially/mock-factory/external/index.md`](./partially/mock-factory/external/index.md) | External Partial Mocking | Übersicht über externes partielles Mocking |

##### Internal __mocks__
| Datei | Thematik | Beschreibung |
|-------|----------|--------------|
| [`partially/mock-factory/external/internal __mocks__/index.md`](./partially/mock-factory/external/internal%20__mocks__/index.md) | Internal __mocks__ Übersicht | Einführung in interne __mocks__ Strukturen |
| [`partially/mock-factory/external/internal __mocks__/hoisted/vi.mocked.md`](./partially/mock-factory/external/internal%20__mocks__/hoisted/vi.mocked.md) | vi.mocked() Hoisting | Verwendung von vi.mocked() in gehoistetem Kontext |
| [`partially/mock-factory/external/internal __mocks__/runtime/doMock.md`](./partially/mock-factory/external/internal%20__mocks__/runtime/doMock.md) | doMock() Runtime | Runtime-Verwendung von doMock() in internen Mocks |

#### Mock Factory (Internal)
| Datei | Thematik | Beschreibung |
|-------|----------|--------------|
| [`partially/mock-factory/internal/importActual.md`](./partially/mock-factory/internal/importActual.md) | vi.importActual() | Importieren echter Module bei partiellem Mocking |
| [`partially/mock-factory/internal/mockObject.md`](./partially/mock-factory/internal/mockObject.md) | Objekt-spezifisches Mocking | Mocking spezifischer Objekteigenschaften und -methoden |

#### Runtime Partial Mocking
| Datei | Thematik | Beschreibung |
|-------|----------|--------------|
| [`partially/runtime/vi.importMock.md`](./partially/runtime/vi.importMock.md) | vi.importMock() Partial | Partielles Mocking mit vi.importMock() zur Laufzeit |

---

### 🕵️ Spy Mocking
Überwachung von Modulen ohne vollständiges Mocking.

#### Internal Spying
| Datei | Thematik | Beschreibung |
|-------|----------|--------------|
| [`spy/internal/vi-mock.md`](./spy/internal/vi-mock.md) | vi.mock() als Spy | Verwendung von vi.mock() für Spying-Zwecke |

---

## 🚀 Best Practices

## ⚡ **Hoisted Mock Strategies** (Kompilierzeit)

### 🏆 **BEST PRACTICE: `vi.mocked()`**
- **Datei:** [`internal __mocks__/hoisted/vi.mocked.md`](./internal%20__mocks__/hoisted/vi.mocked.md)
- **Warum optimal:** Beste Performance, einfachste Syntax, automatisches Hoisting, perfekte TypeScript-Typisierung
- **Ideal für:** Standard-Mocking, maximale Performance, einfache Test-Setups

---

## 🚀 **Runtime Mock Strategies** (Zur Laufzeit)

> **⚠️ WICHTIG:** Runtime-Mocks **MÜSSEN** nur in **Edge-Cases** verwendet werden, wo andere Dependencies ein gemocktes Modul aufrufen und es bei der Hoisted-Variante zu früh gemockt wird. **PRIORISIERE IMMER ZUERST die Hoisted-Mock-Strategien** - nur wenn Hoisting-Konflikte auftreten, wechsle zur Runtime-Variante.

**Für Runtime-Mocking ist `vi.mock()` mit async Callback die moderne, zukunftsorientierte Lösung** - sie bietet bessere TypeScript-Integration, automatisches Cleanup und konsistentere API im Vergleich zu `vi.doMock()`. **Für Hoisted-Mocking bleibt `vi.mocked()` der Gold-Standard** mit optimaler Performance und einfachster Syntax.


### 🥇 **#1 EMPFOHLEN: `vi.mock()` mit Async Callback**
- **Datei:** [`custom __mocks__/runtime/viMock-callback.md`](./custom%20__mocks__/runtime/viMock-callback.md)
- **Warum besser:** Moderne Vitest-Best-Practice, automatisches Cleanup, bessere TypeScript-Integration, konsistentere API
- **Ideal für:** NPM Package FS-Konflikte, komplexe Mock-Factories, dynamische Mocks

### 🥈 **#2 ALTERNATIVE: `vi.doMock()`**
- **Datei:** [`internal __mocks__/runtime/doMock.md`](./internal%20__mocks__/runtime/doMock.md)
- **Warum zweitrangig:** Zwei-Schritt-Prozess, manuelles Cleanup erforderlich, mehr Boilerplate
- **Ideal für:** Legacy-Code, spezielle Edge-Cases, wenn Callback-Approach nicht funktioniert


---

## 📋 **Quick Decision Guide**

| Szenario | Empfohlene Strategie | Begründung |
|----------|---------------------|------------|
| **NPM Package FS-Konflikte** | `vi.mock()` + Callback | Vermeidet Hoisting-Probleme |
| **Dynamische Mock-Konfiguration** | `vi.mock()` + Callback | Flexibelste Runtime-Kontrolle |
| **Standard Unit Tests** | `vi.mocked()` | Beste Performance + einfachste Syntax |
| **Legacy Code Migration** | `vi.doMock()` | Schrittweise Modernisierung möglich |
| **Komplexe Mock Factories** | `vi.mock()` + Callback | Modularste und wartbarste Lösung |


