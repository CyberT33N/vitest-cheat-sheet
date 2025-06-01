
# Manually mocking

<br><br>

### vi.mock()

<details><summary>Click to expand..</summary>

<br><br>

# Example  - fs/promises

```typescript
// test/my-file.test.ts
import * as fsPromises from 'fs/promises';
import { vi, describe, it, expect } from 'vitest';

// Mock das gesamte fs/promises Modul
vi.mock('fs/promises', () => ({
  mkdir: vi.fn(),
  writeFile: vi.fn(),
  readFile: vi.fn(),
  readdir: vi.fn(),
}));

// Typisierte Referenz zum gemockten Modul
const mockedFsPromises = vi.mocked(fsPromises, true);

describe('Dateisystem Operationen', () => {
  it('sollte eine Datei schreiben', async () => {
    const filePath = 'test/output.txt';
    const fileContent = 'Hallo Welt!';

    // Setze die Implementierung für writeFile
    mockedFsPromises.writeFile.mockResolvedValue(undefined);

    // Rufe die Funktion auf, die writeFile verwendet
    await fsPromises.writeFile(filePath, fileContent);

    // Überprüfe, ob writeFile mit den korrekten Argumenten aufgerufen wurde
    expect(mockedFsPromises.writeFile).toHaveBeenCalledWith(filePath, fileContent);
  });

  it('sollte ein Verzeichnis erstellen', async () => {
    const dirPath = 'test/neues-verzeichnis';

    // Setze die Implementierung für mkdir
    mockedFsPromises.mkdir.mockResolvedValue(undefined);

    // Rufe die Funktion auf, die mkdir verwendet
    await fsPromises.mkdir(dirPath, { recursive: true });

    // Überprüfe, ob mkdir mit den korrekten Argumenten aufgerufen wurde
    expect(mockedFsPromises.mkdir).toHaveBeenCalledWith(dirPath, { recursive: true });
  });

  it('sollte den Inhalt einer Datei lesen', async () => {
    const filePath = 'test/input.txt';
    const mockContent = 'Gemockter Dateiinhalt';

    // Setze die Implementierung für readFile
    mockedFsPromises.readFile.mockResolvedValue(mockContent);

    // Rufe die Funktion auf, die readFile verwendet
    const content = await fsPromises.readFile(filePath, 'utf-8');

    // Überprüfe, ob readFile mit den korrekten Argumenten aufgerufen wurde
    expect(mockedFsPromises.readFile).toHaveBeenCalledWith(filePath, 'utf-8');
    // Überprüfe den zurückgegebenen Inhalt
    expect(content).toBe(mockContent);
  });
});
```

<br><br>

# Example - fs (synchron)

<details><summary>Click to expand..</summary>

<br><br>

```typescript
// test/my-sync-file.test.ts
import * as fsSync from 'fs';
import { vi, describe, it, expect } from 'vitest';

// Mock das gesamte fs Modul
vi.mock('fs', () => ({
  existsSync: vi.fn(),
  mkdirSync: vi.fn(),
  writeFileSync: vi.fn(),
  readFileSync: vi.fn(),
}));

// Typisierte Referenz zum gemockten Modul
const mockedFsSync = vi.mocked(fsSync, true);

describe('Synchrone Dateisystem Operationen', () => {
  it('sollte prüfen, ob eine Datei existiert', () => {
    const filePath = 'test/existierende-datei.txt';

    // Setze die Implementierung für existsSync
    mockedFsSync.existsSync.mockReturnValue(true);

    // Rufe die Funktion auf, die existsSync verwendet
    const exists = fsSync.existsSync(filePath);

    // Überprüfe, ob existsSync mit den korrekten Argumenten aufgerufen wurde
    expect(mockedFsSync.existsSync).toHaveBeenCalledWith(filePath);
    // Überprüfe den zurückgegebenen Wert
    expect(exists).toBe(true);
  });

  it('sollte eine Datei synchron schreiben', () => {
    const filePath = 'test/output-sync.txt';
    const fileContent = 'Synchroner Inhalt';

    // Setze die Implementierung für writeFileSync
    mockedFsSync.writeFileSync.mockReturnValue(undefined);

    // Rufe die Funktion auf, die writeFileSync verwendet
    fsSync.writeFileSync(filePath, fileContent);

    // Überprüfe, ob writeFileSync mit den korrekten Argumenten aufgerufen wurde
    expect(mockedFsSync.writeFileSync).toHaveBeenCalledWith(filePath, fileContent);
  });
});
```



</details>


</details>