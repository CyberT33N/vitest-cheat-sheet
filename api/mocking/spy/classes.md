


# Class

## Method

### Public

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













<br><br>
<br><br>


### Privat

<details><summary>Click to expand..</summary>



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

Option1 - `as keyof` :

```typescript
import { describe, it, expect, vi, beforeEach, type MockedObject, type MockedFunction, MockInstance } from 'vitest'

describe('getOrCreateIndex()', () => {
    describe('✅ Positive Tests', () => {
        let createIndexAndWaitSpy: MockInstance

        const mockIndexModel: IndexModel = {
            name: TEST_DATA.indexName,
            dimension: TEST_DATA.dimension,
            metric: 'cosine',
            host: 'test-host',
            spec: { serverless: { cloud: TEST_DATA.cloudProvider, region: TEST_DATA.region } },
            status: { ready: true, state: 'Ready' },
            vectorType: 'float'
        }

        beforeEach(() => {
            createIndexAndWaitSpy = vi.spyOn(service, '_createIndexAndWait' as keyof PineconeService)
                .mockResolvedValue(mockIndexInstance)
        })

        // eslint-disable-next-line max-len
        it.only('sollte _createIndexAndWait aufrufen, wenn _describeIndex fehlschlägt (Index nicht vorhanden)', async() => {
            mockPineconeInstance.describeIndex
                .mockRejectedValueOnce(new Error('Index not found'))
                .mockResolvedValueOnce(mockIndexModel)

            const indexOptions = createTestIndexOptions()
            const result = await service.getOrCreateIndex(indexOptions)

            expect(createIndexAndWaitSpy).toHaveBeenCalledWith(indexOptions)
            expect(createIndexAndWaitSpy).toHaveBeenCalledTimes(1)
            expect(result).toBeDefined()
        })
    })
})
```





Option2 - `prototype` :

```typescript
describe('prax', () => {
    let spyOnGetPrax: MockInstance

    beforeEach(() => {
        spyOnGetPrax = vi.spyOn(DampsoftService.prototype, '_getPrax').mockReturnValue(
            ['PRAX1', 'PRAX2']
        )
    })

    it('sollte Array-basierte DBF-Pfade basierend auf prax-Verzeichnissen initialisieren', () => {
        const service = new DampsoftService()
        expect(spyOnGetPrax).toHaveBeenCalledOnce()
        expect(service.befundDBPath[0]).toMatch(/.*\/DS\/daten\/PRAX\d+\/BEFUND\.DBF/)
        expect(service.hkpPlanDBPath[0]).toMatch(/.*\/DS\/daten\/PRAX\d+\/HKPPLAN\.DBF/)
        expect(service.psiDBPath[0]).toMatch(/.*\/DS\/daten\/PRAX\d+\/PSI\.DBF/)
        expect(service.rechnungDBPath[0]).toMatch(/.*\/DS\/daten\/PRAX\d+\/RECHNUNG\.DBF/)
    })
})
```

</details>




