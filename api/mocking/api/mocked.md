
# `vi.mocked()`

## Optionen**

### **`partial: true` - Partielle Mock-Erstellung**

```typescript
// ❌ OHNE partial: true
const mockedAxios = vi.mocked(axios)
// Problem: Vitest erwartet, dass ALLE Eigenschaften von axios gemockt sind
// Fehler: Wenn axios 50+ Eigenschaften hat, müssen alle definiert werden

// ✅ MIT partial: true  
const mockedAxios = vi.mocked(axios, { partial: true })
// Lösung: Nur die Eigenschaften mocken, die wir tatsächlich verwenden
```

**Was passiert:**
- **Ohne `partial`**: Vitest erstellt einen **vollständigen Mock** - alle Eigenschaften müssen definiert sein
- **Mit `partial`**: Vitest erstellt einen **partiellen Mock** - nur verwendete Eigenschaften werden gemockt
- **Unser Nutzen**: Wir brauchen nur `axios.get`, nicht alle 50+ axios-Eigenschaften








<br><br>
<br><br>

### **`deep: true` - Tiefes Mock-Verhalten**

```typescript
// ❌ OHNE deep: true
const mockedAxios = vi.mocked(axios, { partial: true })
mockedAxios.get.mockResolvedValueOnce() // ❌ FEHLER: mockResolvedValueOnce existiert nicht

// ✅ MIT deep: true
const mockedAxios = vi.mocked(axios, { partial: true, deep: true })
mockedAxios.get.mockResolvedValueOnce() // ✅ FUNKTIONIERT: Mock-Methoden verfügbar
```

**Was passiert:**
- **Ohne `deep`**: Nur **oberste Ebene** wird gemockt (`axios` selbst)
- **Mit `deep`**: **Alle verschachtelten Eigenschaften** werden automatisch zu Mock-Funktionen
- **Unser Nutzen**: `axios.get` wird automatisch zu einer `vi.fn()` mit allen Mock-Methoden

## **🎯 In unserem KONKRETEN Beispiel:**

### **Ohne die Optionen (Probleme):**
```typescript
// ❌ PROBLEMATISCH
const mockedAxios = vi.mocked(axios)

// Problem 1: TypeScript-Fehler wegen fehlender Eigenschaften
// Problem 2: mockedAxios.get hat keine Mock-Methoden
// Problem 3: Muss alle axios-Eigenschaften definieren
```

### **Mit den Optionen (Lösung):**
```typescript
// ✅ OPTIMAL
const mockedAxios = vi.mocked(axios, { partial: true, deep: true })

// ✅ partial: true → Nur axios.get muss definiert sein
// ✅ deep: true → axios.get hat automatisch .mockResolvedValueOnce()
// ✅ Typsicher → Keine any-Casts nötig
```

## **📊 Vergleich der Ansätze:**

| Ansatz | TypeScript-Sicherheit | Mock-Methoden | Code-Aufwand |
|--------|----------------------|---------------|--------------|
| `vi.mocked(axios)` | ❌ Fehler | ❌ Keine | 🔴 Hoch |
| `vi.mocked(axios, {partial: true})` | ✅ OK | ❌ Keine | 🟡 Mittel |
| `vi.mocked(axios, {partial: true, deep: true})` | ✅ Perfekt | ✅ Alle | 🟢 Minimal |

**Fazit:** Die Kombination `{ partial: true, deep: true }` ist der **Enterprise-Standard** für moderne TypeScript-Mock-Architekturen! 🚀