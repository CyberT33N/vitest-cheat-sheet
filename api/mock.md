
### 🔍 Methoden zur direkten Prüfung (`.mock`-Objekt)

```ts
// Wurde der Spy überhaupt aufgerufen?
const wasCalled = patientServiceSpy.mock.calls.length > 0;

// Wie oft wurde er aufgerufen?
const callCount = patientServiceSpy.mock.calls.length;

// Mit welchen Argumenten beim 1. Call?
const firstCallArgs = patientServiceSpy.mock.calls[0];

// Mit welchen Argumenten beim letzten Call?
const lastCallArgs = patientServiceSpy.mock.calls.at(-1);

// Ergebnis des 1. Calls?
const firstCallResult = patientServiceSpy.mock.results[0]; // { type: 'return', value: ... }
```

---

### 💡 Beispiel

```ts
if (patientServiceSpy.mock.calls.length === 0) {
  console.warn('getPatients wurde nicht aufgerufen');
}

for (const call of patientServiceSpy.mock.calls) {
  console.log('Args:', call);
}

console.log('Return value beim ersten Aufruf:', patientServiceSpy.mock.results[0]?.value);
```

---

### 🧠 Pro-Tipp: Zugriff auf `this` und Instanzen

```ts
// Falls Methode auf einer Klasse mit `this` lief
patientServiceSpy.mock.instances // Alle `this`-Kontexte
```

---

### 🧬 Bonus: Use-Case ohne `expect`

```ts
if (callCount > 3) {
  doSomethingCool();
}
```
