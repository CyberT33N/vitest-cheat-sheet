# Methods – Public

<details><summary>Click to expand..</summary>

Option1 - `ClassName.prototype` :

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

</details>
