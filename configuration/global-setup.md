# globalSetup

Die `globalSetup`-Option in der Vitest-Konfiguration definiert den Pfad zu einer Datei, die zu Beginn der Testsuite einmalig ausgeführt wird.

Offizielle Dokumentation: [https://vitest.dev/config/#globalsetup](https://vitest.dev/config/#globalsetup)

> **Wichtig**: Diese Datei sollte verwendet werden, um beispielsweise einen Express-Server zu starten oder ähnliche Setup-Aufgaben durchzuführen. Wenn globale Variablen vor jedem Test deklariert werden sollen, sollte stattdessen `setupFiles` verwendet werden. Wenn `global.test=123` in der `globalSetup`-Datei in der exportierten setup()-Funktion gesetzt wird, wäre der Wert in den Tests undefined.

## Beispiel

```typescript
export async function setup() {
    server.start()
}

export async function teardown() {
    server.close()
}
```

Eine Global-Setup-Datei kann entweder benannte Funktionen `setup` und `teardown` exportieren oder eine Standardfunktion, die eine Teardown-Funktion zurückgibt (siehe [Beispiel](https://github.com/vitest-dev/vitest/blob/main/test/global-setup/vitest.config.ts)). 