
# Zugriff auf private Properties/Methoden in TypeScript Tests

In TypeScript sind private Properties und Methoden standardmäßig von außerhalb der Klasse nicht zugänglich. Dies dient der Kapselung und soll sicherstellen, dass Klassen nur über ihre öffentliche API interagieren. In Unit-Tests kann es jedoch manchmal notwendig sein, auf private Mitglieder zuzugreifen, um interne Zustände zu überprüfen oder private Logik direkt zu testen, ohne die öffentliche API zu verändern oder die Kapselung im Produktionscode aufzuweichen.

Diese Regel beschreibt die präferierten und alternativen Methoden, um in Testumgebungen auf private Mitglieder zuzugreifen.

## ❗ Critical Rules

*   **PRÄFERIERTER ANSATZ:** Verwende die **Bracket Notation** (`instance['_privateMemberName']`) mit deaktivierter `dot-notation` ESLint-Regel, um auf private Properties und Methoden zuzugreifen. Dies ist der sauberste und typsicherste Weg, wenn die ESLint-Konfiguration angepasst werden kann. **Bei Helper-Funktionen MUSS** eine **Existenzprüfung** (`if (service instanceof ClassName)`) durchgeführt werden, bevor auf private Properties zugegriffen wird.
*   **ALTERNATIV ANSATZ:** Wenn die ESLint-Konfiguration nicht geändert werden kann oder der Bracket Notation Ansatz aus anderen Gründen nicht praktikabel ist, verwende `Reflect.get(instance, 'privatePropertyName')` für Properties und `(instance as any)['_privateMethodName'](...args)` oder `Reflect.apply((instance as any)['_privateMethodName'], instance, args)` für Methoden. Beachte, dass dieser Ansatz eine Typ-Assertion (`as any`) erfordert.
*   **MUST:** Dieser Ansatz sollte **ausschließlich** in Unit-Tests verwendet werden.
*   **CONSIDER:** Ob der direkte Test einer privaten Methode oder Property wirklich notwendig ist. Oft kann das Verhalten auch über die öffentliche API getestet werden. Direkte Tests privater Mitglieder können Tests fragiler gegenüber Refactorings machen.
*   **MUST NOT:** Verwende diese Ansätze im Produktionscode.

## 🎯 Präferierter Ansatz: Bracket Notation

Die **Bracket Notation** ist die sauberste und typsicherste Methode zum Testen privater Mitglieder in TypeScript, vorausgesetzt, die ESLint-Konfiguration erlaubt dies.

### ✅ Vorteile:
- ✅ Typsicher (mit korrekter ESLint-Konfiguration oder expliziter Typ-Assertion für verschachtelte Strukturen)
- ✅ Sauber - Weniger komplexe Type Assertions im direkten Aufruf (außer bei tief verschachtelten Strukturen)
- ✅ Wartbar - Einfach zu verstehen
- ✅ Linterfrei (mit korrekter ESLint-Konfiguration)

### 🔧 ESLint-Konfiguration (Erforderlich für Bracket Notation ohne Type Assertion bei direkten Membern)

**Wichtig:** Die `dot-notation` Regel muss in der ESLint-Konfiguration deaktiviert werden:

```javascript
// eslint.config.mjs
export default tseslint.config(
    // ... andere Konfigurationen

    // ===== ESLINT CORE RULES CUSTOMIZATION =====
    {
        rules: {
            // ... andere Regeln
            'dot-notation': 'off' // Disabled to allow bracket notation for private method testing
        }
    },

    // ===== ADDITIONAL TYPESCRIPT RULES =====
    {
        rules: {
            // ... andere Regeln
            '@typescript-eslint/dot-notation': 'off', // Disabled to allow bracket notation for private method testing
        }
    }
)
```

### 📝 Beispiel-Test (Bracket Notation)

```typescript
// Beispiel-Service mit privater Methode und Property
class UserService {
    private _isValid: boolean = false; // Private Property
    private _details = { userType: 'guest', permissions: ['read'] }; // Verschachtelte private Property

    private _validateEmail(email: string): boolean {
        this._isValid = email.includes('@'); // Setzt private Property
        return this._isValid;
    }

    public createUser(email: string): any { // Rückgabetyp angepasst für Beispiel
        if (!this._validateEmail(email)) {
            throw new Error('Invalid email');
        }
        // ... weitere Logik
        return { email, isValid: this._isValid, type: this._details.userType }; // Beispiel Rückgabe
    }

    private async _someAsyncPrivateMethod(data: string): Promise<string> {
        return Promise.resolve(data.toUpperCase());
    }
}

// Test für die private Methode und Property
describe('UserService', () => {
    let service: UserService;

    beforeEach(() => {
        service = new UserService();
    });

    describe('_validateEmail() - Private Method Tests (Bracket Notation)', () => {
        it('sollte gültige E-Mail-Adressen akzeptieren und _isValid setzen', () => {
            // ✅ Bracket Notation für private Methode
            const result = service['_validateEmail']('test@example.com');
            expect(result).toBe(true);
            // ✅ Bracket Notation für private Property lesen
            expect(service['_isValid']).toBe(true);
        });

        it('sollte ungültige E-Mail-Adressen ablehnen und _isValid setzen', () => {
            // ✅ Bracket Notation für private Methode
            const result = service['_validateEmail']('invalid-email');
            expect(result).toBe(false);
            // ✅ Bracket Notation für private Property lesen
            expect(service['_isValid']).toBe(false);
        });

        it('sollte bei async privaten Methoden funktionieren (Bracket Notation)', async () => {
            // ✅ Auch für async Methoden
            const result = await service['_someAsyncPrivateMethod']('data');
            expect(result).toBe('DATA');
        });
    });

    describe('Private Property Access Tests (Bracket Notation)', () => {
        it('sollte private Properties lesen können', () => {
            // ✅ Private Properties lesen
            expect(service['_isValid']).toBe(false); // Initialwert
        });

        it('sollte private Properties setzen können', () => {
            // ✅ Private Properties setzen
            service['_isValid'] = true;
            expect(service['_isValid']).toBe(true);
        });

        it('sollte auf verschachtelte private Properties zugreifen können', () => {
            // ✅ Direkter Zugriff auf verschachtelte private Property
            expect(service['_details'].userType).toBe('guest');

            // Ändern der verschachtelten Property
            service['_details'].userType = 'admin';
            expect(service['_details'].userType).toBe('admin');
        });
    });

    describe('Advanced Private Method Testing', () => {
        class ComplexService {
            private _namespace: string = 'test-namespace';
            private _config: { apiKey?: string } = {};

            private async _processComplexData(data: { id: number, name: string }, options: { validate: boolean }): Promise<{ processed: boolean, data: any }> {
                if (options.validate && !data.name) {
                    throw new Error('Name is required');
                }
                return { processed: true, data };
            }
        }
        let complexService: ComplexService;
        beforeEach(() => {
            complexService = new ComplexService();
        });

        it('sollte private Properties lesen können', () => {
            // ✅ Private Properties
            expect(complexService['_namespace']).toBe('test-namespace');
        });

        it('sollte private Properties setzen können', () => {
            // ✅ Private Properties setzen
            complexService['_config'] = { apiKey: 'test-key' };
            expect(complexService['_config'].apiKey).toBe('test-key');
        });

        it('sollte komplexe private Methoden mit Parametern testen', async () => {
            const mockData = { id: 1, name: 'Test' };

            // ✅ Private Methode mit komplexen Parametern
            const result = await complexService['_processComplexData'](mockData, { validate: true });

            expect(result).toMatchObject({
                processed: true,
                data: mockData
            });
        });
    });

    describe('Helper Function Tests mit Existenzprüfung', () => {
        it('sollte Helper-Funktion mit Existenzprüfung verwenden', () => {
            // ✅ Helper-Funktion mit korrekter Existenzprüfung
            testConstructorDefaults(service);
        });
    });
});

/**
 * Helper function to test constructor initialization with default values
 * @param service - The service to test
 */
export function testConstructorDefaults(service: Readonly<UserService>): void {
    if (service instanceof UserService) {
        expect(service).toBeInstanceOf(UserService)
        expect(service['_isValid']).toBe(false)
        expect(service['_details'].userType).toBe('guest')
        expect(service['_details'].permissions).toEqual(['read'])
    }
}
```

## 🏗️ Alternativ Ansatz: Reflect

Wenn die Bracket Notation aufgrund von ESLint-Einschränkungen oder anderen Gründen nicht verwendet werden kann, bietet die `Reflect`-API eine Alternative.

### 📝 Beispiel-Test (Reflect)

```typescript
// Dummy Class mit privater Property und Methode
class DataService {
    private _data: any = null; // Private Property
    private _nested = { info: "initial" };

    private _processData(input: any): any {
        this._data = input; // Setzt private Property
        return { processed: input };
    }

    public processExternal(input: any): any {
        return this._processData(input);
    }
}

describe('DataService', () => {
    let service: DataService;

    beforeEach(() => {
        service = new DataService();
    });

    describe('_processData() - Private Method Tests (Reflect)', () => {
        it('sollte private Methode aufrufen können', () => {
            // ✅ KORREKT: Aufruf einer privaten Methode mittels Reflect.apply
            const result = Reflect.apply((service as any)['_processData'], service, ['hello']); // Type Assertion nötig
            expect(result).toEqual({ processed: 'hello' });
            // ✅ KORREKT: Zugriff auf private Property mittels Reflect.get
            expect(Reflect.get(service, '_data')).toBe('hello');
        });
    });

    describe('Private Property Access Tests (Reflect)', () => {
        it('sollte private Property lesen können', () => {
            // ✅ KORREKT: Zugriff auf private Property mittels Reflect.get
            expect(Reflect.get(service, '_data')).toBe(null); // Initialwert
        });

        it('sollte private Property setzen können', () => {
            // ✅ KORREKT: Setzen einer privaten Property mittels Reflect.set
            Reflect.set(service, '_data', { id: 1 });
            expect(Reflect.get(service, '_data')).toEqual({ id: 1 });
        });

        it('sollte auf verschachtelte private Properties zugreifen können', () => {
            // ✅ KORREKT: Zugriff auf verschachtelte private Property mittels Reflect.get
            const nestedObject = Reflect.get(service, '_nested') as { info: string };
            expect(nestedObject.info).toBe('initial');

            // Ändern der verschachtelten Property
            nestedObject.info = 'updated';
            // Überprüfen, ob die Änderung erfolgreich war (optional, da Reflect.set hier nicht direkt verwendet wird)
            const updatedNestedObject = Reflect.get(service, '_nested') as { info: string };
            expect(updatedNestedObject.info).toBe('updated');
        });
    });
});
```

## ⚠️ Wichtige Hinweise

1.  **ESLint-Konfiguration erforderlich:** Für den präferierten Bracket Notation Ansatz muss die `dot-notation` Regel deaktiviert werden.
2.  **Nur für Tests verwenden:** Direkter Zugriff auf private Mitglieder sollte **ausschließlich** in Testdateien erfolgen.
3.  **Dokumentation:** Kommentiere Tests, die private Mitglieder verwenden, entsprechend.
4.  **Alternative Teststrategien:** Wenn möglich, teste das Verhalten über die öffentliche API anstatt private Mitglieder direkt aufzurufen. Dies führt oft zu robusteren Tests.

## 🎯 Best Practice Template (Bracket Notation)

```typescript
describe('ClassName - Private Member Tests', () => {
    let instance: ClassName; // Ersetze ClassName mit dem tatsächlichen Klassennamen

    beforeEach(() => {
        instance = new ClassName(); // Ersetze ClassName mit dem tatsächlichen Klassennamen
    });

    describe('_privateMethodName()', () => {
        it('sollte [erwartetes Verhalten] bei [Bedingung]', () => {
            // Arrange
            const input = 'test-data';
            const expectedValue = 'TEST-DATA'; // Beispiel

            // Act
            const result = instance['_privateMethodName'](input); // Bracket Notation

            // Assert
            expect(result).toBe(expectedValue);
        });
    });

    describe('_privatePropertyName', () => {
        it('sollte den korrekten Wert haben', () => {
            // Arrange
            // ... setze Zustand, der die private Property beeinflusst ...
            const expectedValue = 'initial-value'; // Beispiel

            // Assert
            expect(instance['_privatePropertyName']).toBe(expectedValue); // Bracket Notation
        });
    });

    describe('_privateNestedObject.property', () => {
        it('sollte den korrekten Wert der verschachtelten Property haben', () => {
            // Arrange
            const expectedValue = 'nested-value';
            // Annahme: instance hat ein privates Objekt _privateNestedObject mit einer Property 'property'
            instance['_privateNestedObject'].property = expectedValue;

            // Assert
            expect(instance['_privateNestedObject'].property).toBe(expectedValue);
        });
    });

    describe('Helper Function Template', () => {
        it('sollte Helper-Funktion mit Existenzprüfung verwenden', () => {
            // ✅ Helper-Funktion mit korrekter Existenzprüfung
            testHelperFunction(instance);
        });
    });
});

/**
 * Helper function to test private members with existence check
 * @param service - The service to test
 */
export function testHelperFunction(service: Readonly<ClassName>): void {
    if (service instanceof ClassName) {
        expect(service).toBeInstanceOf(ClassName)
        expect(service['_privatePropertyName']).toBe('expected-value')
        expect(service['_privateNestedObject'].property).toBe('expected-nested-value')
    }
});
```

## 🤔 Begründung

Der direkte Zugriff auf private Mitglieder in Tests ist ein kontroverses Thema. Befürworter argumentieren, dass es die Testbarkeit erhöht und sicherstellt, dass interne Logik korrekt funktioniert, ohne die öffentliche API zu belasten. Kritiker weisen darauf hin, dass solche Tests eng an die Implementierungsdetails gekoppelt sind und bei Refactorings leicht brechen.

Die **Bracket Notation** ist, wenn durch ESLint erlaubt, der präferierte Weg, da sie syntaktisch sauberer ist und sich besser in den TypeScript-Workflow integriert. **Bei Helper-Funktionen ist eine Existenzprüfung mit `instanceof` zwingend erforderlich**, um sicherzustellen, dass das Objekt existiert und vom erwarteten Typ ist. Die `Reflect`-API bietet eine Alternative, ist aber oft weniger elegant.

Es ist wichtig, sich der Implikationen bewusst zu sein und diesen Ansatz sparsam und überlegt einzusetzen. Die primäre Teststrategie sollte immer darin bestehen, das Verhalten über die öffentliche API zu verifizieren.

**🏆 Fazit:** Die Bracket Notation mit deaktivierter `dot-notation` ESLint-Regel und korrekter Existenzprüfung in Helper-Funktionen ist die beste Lösung für den direkten Zugriff auf private Mitglieder in TypeScript-Tests!