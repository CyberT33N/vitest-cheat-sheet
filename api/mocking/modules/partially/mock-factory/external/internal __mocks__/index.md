# Internal `__mocks__` - Automatische Out-of-the-Box Mocks

## 🎯 **Zweck & Verwendung**

Dieser Ordner repräsentiert die **Out-of-the-Box Variante** von Vitest - automatisches Mocking über den **Route-Level `__mocks__` Ordner**. Diese Mocks werden **automatisch** von Vitest verwendet, wenn `vi.mock()` **ohne Callback** aufgerufen wird.

## 🚀 **Automatisches Mocking - Wie es funktioniert:**

### 📁 **Route-Level `__mocks__` Ordner:**
```
src/
├── services/
│   └── userService.ts
├── __mocks__/          # ← Route-Level Mock-Ordner
│   └── userService.ts  # ← Automatischer Mock
└── utils/
    ├── fileHandler.ts
    └── __mocks__/      # ← Route-Level Mock-Ordner
        └── fileHandler.ts
```

### ⚡ **Automatische Verwendung:**
```typescript
// ✅ OUT-OF-THE-BOX - Verwendet automatisch __mocks__/userService.ts
vi.mock('./services/userService') // KEIN Callback nötig!

// ✅ Vitest findet und lädt automatisch:
// src/__mocks__/userService.ts oder
// src/services/__mocks__/userService.ts
```

## 🔧 **Vorteile der automatischen Mocks:**

### ✅ **Einfachste Verwendung:**
- **Kein Callback-Code nötig**
- **Keine expliziten Imports**
- **Vitest macht alles automatisch**

### ⚡ **Beste Performance:**
- **Zur Compile-Zeit gemockt** (Hoisted)
- **Minimaler Overhead**
- **Keine Runtime-Mock-Erstellung**

### 📚 **Standard Vitest Workflow:**
```typescript
// 1. Mock-Datei erstellen: src/__mocks__/fs.ts
export const readFile = vi.fn()
export const writeFile = vi.fn()
export default { readFile, writeFile }

// 2. In Test verwenden - OHNE Callback
vi.mock('fs') // ← Automatisch src/__mocks__/fs.ts geladen

// 3. Typisierte Referenz erhalten
const mockedFs = vi.mocked(fs, { deep: true })
```

## 🎯 **Wann Route-Level `__mocks__` verwenden:**

- **Standard Unit Tests** ohne spezielle Runtime-Anforderungen
- **Maximale Performance** ist wichtig
- **Einfache Mock-Konfiguration** ausreichend
- **Keine NPM Package Konflikte** vorhanden
- **Team bevorzugt** Out-of-the-Box Lösungen

## ⚠️ **Limitationen:**

- **Hoisting-Konflikte:** Bei NPM Package Dependencies möglich
- **Weniger flexibel:** Schwieriger für komplexe Runtime-Konfiguration  
- **Statisch:** Keine dynamische Mock-Erstellung zur Laufzeit
