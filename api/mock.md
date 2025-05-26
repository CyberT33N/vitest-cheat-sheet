# Class

## Method
```typescript
describe('Patients', () => {
    let patientServiceSpy: MockInstance<PatientService['getPatients']>

    beforeEach(() => {
        patientServiceSpy = vi.spyOn(PatientService.prototype, 'getPatients')
    })

    it.only('should return a filtered list of patients when patientId is provided', async() => {
        // Assuming '1' is a patientId that might exist or return an empty list,
        // which is fine for testing the filter mechanism.
        const patientIdToFilter = '1' 
        const response = await testServer.apiClient.get<IPatientsResponse>(
            `${API_PATH}/patients?patientId=${patientIdToFilter}`
        )

        expect(patientServiceSpy).toHaveBeenCalledWith(false, { patientId: patientIdToFilter })
        
        expect(response.status).toBe(200)
        expect(response.data).toHaveProperty('success', true)
        expect(response.data).toHaveProperty('patients')
        expect(Array.isArray(response.data.patients)).toBe(true)
        // Further assertions could be added here if we know specific data about patient '1'
        // For example, if patient '1' exists, we could check:
        // if (response.data.patients.length > 0) {
        //   expect(response.data.patients[0]).toHaveProperty('PATNR', patientIdToFilter);
        // }
    })
})
```






<br><br>
________
________
<br><br>



<details><summary>Click to expand..</summary>
</details>





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
