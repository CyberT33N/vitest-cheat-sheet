# Strategische Einordnung: Internes `__mocks__`-Verzeichnis

Diese Seite analysiert die Verwendung eines "internen" `__mocks__`-Verzeichnisses. Dabei wird der Mock-Ordner direkt neben der zu testenden Quelldatei platziert, anstatt auf der Root-Ebene.

### ⚠️ Wichtige Vorbemerkung: Meist überflüssig durch Standard-Features

Die hier beschriebene Methode ist technisch zwar möglich, aber in den meisten Fällen **überflüssig und nicht die empfohlene Vorgehensweise**. Vitest bietet eine eingebaute, überlegene "out-of-the-box"-Lösung für automatisches Hoisted Mocking über ein `__mocks__`-Verzeichnis auf der Root-Ebene des Projekts.

Dieses interne Muster sollte nur mit Bedacht und als bewusste architektonische Entscheidung für Nischenfälle eingesetzt werden.

---

## 📁 Verzeichnisstruktur & Beispiele

```
.
├── hoisted/
│   └── vi.mocked.md
├── runtime/
│   └── doMock.md
└── index.md
```

---

## Hierarchie der Mocking-Strategien

Für das Hoisted Mocking (also Mocks zur Kompilierzeit) sollte folgende Priorisierung gelten:

### 🥇 1. Priorität (Standard): Root-Level `__mocks__` (Out-of-the-Box)
- **Beschreibung:** Ein `__mocks__`-Ordner wird auf der obersten Ebene Ihres Projekts (z.B. im Root oder im `test/`-Verzeichnis) angelegt. Vitest findet und verwendet die Mocks darin automatisch, sobald `vi.mock('modul-name')` aufgerufen wird.
- **Vorteil:** Dies ist die **Best Practice** von Vitest. Sie ist sauber, trennt den Quellcode klar von den Test-Artefakten und erfordert keine komplexe Konfiguration.
- **Wann verwenden:** **Für 95% aller Anwendungsfälle**, bei denen ein Modul einfach und vollständig durch einen Mock ersetzt werden soll.

### 🥈 2. Priorität (Fallback): Internes `__mocks__`-Verzeichnis (Diese Methode)
- **Beschreibung:** Ein `__mocks__`-Ordner wird direkt neben die Quelldatei gelegt, die er mockt.
- **Vorteil:** Der einzige, geringfügige Vorteil ist die direkte räumliche Nähe des Mocks zum Quellcode, was eine rein organisatorische Präferenz sein kann.
- **Wann verwenden:** Nur dann, wenn es sich um eine **extrem simple Service-Datei** mit einer **simplen, flachen Struktur** handelt und man den Mock bewusst co-lokalisieren möchte. In fast allen Szenarien ist der Standard-Ansatz (Nr. 1) überlegen.
- **Referenz-Dateien:**
    - [`hoisted/vi.mocked.md`](./hoisted/vi.mocked.md): Zeigt die Kompilierzeit-Variante.
    - [`runtime/doMock.md`](./runtime/doMock.md): Zeigt die Laufzeit-Variante mit `vi.doMock()`.

---

## Abgrenzung zu komplexen Fällen

### Wann ist diese Methode ungeeignet?
Sobald die Struktur des zu mockenden Moduls komplexer wird, ist dieser Ansatz nicht mehr ausreichend.

-   **Komplexe Struktur:** Wenn ein Modul instanzbasiert ist, mehrere verkettete Methoden hat oder eine feingranulare Kontrolle über verschiedene Mock-Zustände erfordert (z.B. ein SDK wie `@pinecone-database/pinecone`).

### Was ist die Lösung für komplexe Strukturen?
-   **Custom Mock-Factory:** Für komplexe Fälle **MUSS** eine **Custom Mock-Factory** mit einem **Runtime-Mock** (z.B. `vi.mock` mit asynchronem Callback) verwendet werden. Dieser Ansatz bietet die nötige Flexibilität und Kontrolle, die für komplexe SDKs unerlässlich ist.

## Fazit

| Strategie | Komplexität | Wann verwenden? | Priorität |
| :--- | :--- | :--- | :--- |
| **Root-Level `__mocks__`** | Einfach | **Standardfall.** Einfache, zustandslose Module. | 🥇 **Hoch** |
| **Custom Mock-Factory (Runtime)** | Komplex | **Komplexe, instanzbasierte SDKs.** | 🏆 **(Für Komplexität)** |
| **Internes `__mocks__`** | Einfach | **Nischenfall.** Nur für extrem simple Services, wenn Co-Lokalisierung gewünscht ist. | 🥉 **Niedrig** |

**Regel:** Beginnen Sie immer mit dem Standard-Ansatz (Root-Level `__mocks__`). Wechseln Sie nur dann auf eine Custom Mock-Factory, wenn die Komplexität es erfordert. Das interne `__mocks__`-Verzeichnis ist eine seltene Ausnahme, keine Regel.
