# Snapshots

Vitest unterstützt Snapshot-Tests, um zu überprüfen, ob die Ausgabe von Komponenten oder Funktionen zwischen verschiedenen Test-Läufen konsistent bleibt.

Offizielle Dokumentation: [https://vitest.dev/guide/snapshot](https://vitest.dev/guide/snapshot)

## Beispiel

```typescript
import { expect, it } from 'vitest'

it('toUpperCase', () => {
  const result = toUpperCase('foobar')
  expect(result).toMatchSnapshot()
})
```

Bei der ersten Ausführung dieses Tests erstellt Vitest eine Snapshot-Datei, die wie folgt aussieht:

```typescript
// Vitest Snapshot v1, https://vitest.dev/guide/snapshot.html
exports['toUpperCase 1'] = '"FOOBAR"'
```

Das Snapshot-Artefakt sollte zusammen mit den Code-Änderungen committet und im Rahmen des Code-Review-Prozesses überprüft werden. Bei nachfolgenden Testläufen vergleicht Vitest die generierte Ausgabe mit dem vorherigen Snapshot. Wenn sie übereinstimmen, wird der Test bestanden. Wenn sie nicht übereinstimmen, hat der Testläufer entweder einen Fehler im Code gefunden, der behoben werden sollte, oder die Implementierung hat sich geändert und der Snapshot muss aktualisiert werden. 