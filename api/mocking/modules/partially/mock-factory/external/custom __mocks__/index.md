# Custom `__mocks__` - Manuelle Mock-Factories

## 🎯 **Zweck & Verwendung**

Dieser Ordner enthält **manuelle Mock-Factory-Dateien**, die **explizit in `vi.mock()` Callbacks importiert** werden. Diese Mocks werden **NICHT automatisch** von Vitest verwendet, sondern sind für **programmatische Mock-Konfiguration** gedacht.

## 📁 **Warum separater `custom __mocks__` Ordner?**

### ✅ **Semantische Trennung:**
- **`__mocks__/` (Route-Level)** = Automatisches Mocking (vi.mock() ohne Callback)
- **`custom __mocks__/`** = Manuelle Mock-Factories (vi.mock() mit Callback + Import)

### 🚫 **Verwirrung vermeiden:**
- Wenn Mock-Factories im Route-Level `__mocks__/` liegen, könnte man denken, sie werden automatisch verwendet
- Das wäre irreführend, da diese explizit mit Callback + Import arbeiten

### 🔧 **Klare Code-Intention:**
```typescript
// ✅ KORREKT - Expliziter Import aus custom __mocks__
vi.mock('fs', async () => {
    const { mockFsModule } = await import('@test/__mocks__/fs.js')
    return mockFsModule()
})

// ❌ VERWIRREND - Import aus Route-Level __mocks__
vi.mock('fs', async () => {
    const { mockFsModule } = await import('../__mocks__/fs.js') // Sieht aus wie Auto-Mock
    return mockFsModule()
})
```

## 🏗️ **Typische Anwendungsfälle:**

- **NPM Package Konflikte:** FS-Module die zu früh gemockt werden
- **Komplexe Mock-Factories:** Dynamische Mock-Konfiguration
- **Runtime-Mocking:** Wenn Hoisted-Mocks Probleme verursachen
- **Modulare Test-Setups:** Wiederverwendbare Mock-Komponenten

## 📋 **Best Practices:**

1. **Explizite Benennung:** Dateinamen sollten Mock-Zweck klar machen
2. **Factory-Pattern:** Exportiere Mock-Factory-Funktionen, nicht direkte Mocks
3. **TypeScript-Support:** Vollständige Typisierung für bessere DX
4. **Dokumentation:** Kommentiere Mock-Verhalten für Team-Verständnis
