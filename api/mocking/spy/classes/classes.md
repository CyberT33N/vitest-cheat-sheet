


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

Option1 - `as keyof` (Statischer Import erforderlich) :

```typescript
import { describe, it, expect, vi, beforeEach, type MockedObject, type MockedFunction, MockInstance } from 'vitest'
// ✅ WICHTIG: Statischer Import der Service-Klasse erforderlich für 'as keyof'
import { DampsoftService } from '@/main/services/dampsoft/DampsoftService.ts'

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

**Alternative für Edge-Cases: Runtime-Import mit erweiterten Typen**

Für Szenarien mit `vi.doMock()` oder anderen Runtime-Imports, wo kein statischer Import möglich ist:

```typescript
import { describe, it, expect, vi, beforeEach, type MockInstance } from 'vitest'

    describe('DampsoftService', () => {
        // eslint-disable-next-line @typescript-eslint/naming-convention
        let DampsoftService: typeof import('@/main/services/dampsoft/DampsoftService.ts').DampsoftService
        let service: InstanceType<typeof DampsoftService>

        beforeEach(async() => {
            ;({ DampsoftService } = await import('@/main/services/dampsoft/DampsoftService.ts'))
            service = new DampsoftService()
        })

        describe('xxx', () => {
            let spyOnGetPrax: MockInstance

            beforeEach(() => {
                    type ExtendedServiceType = InstanceType<typeof DampsoftService> & { 
                        // eslint-disable-next-line @typescript-eslint/naming-convention
                        _getPrax: () => string[] 
                    }

                    spyOnGetPrax = vi.spyOn<ExtendedServiceType, '_getPrax'>(
                        DampsoftService.prototype as ExtendedServiceType,
                        '_getPrax'
                    ).mockReturnValue(['PRAX1', 'PRAX2'])
            })

            it('sollte private Methode korrekt mocken', () => {
                expect(spyOnGetPrax).toHaveBeenCalled()
            })
        })
    })
```



For extended classes you may have to check this:

<details><summary>Click to expand..</summary>

# TypeScript `vi.spyOn()` Problem: Private Methoden in vererbten Klassen

## Problem-Zusammenfassung

**Fehler:** `Type '"_getPrax"' is not assignable to parameter of type "methodName1" | "methodName2" | ...`

## Warum funktioniert es in PineconeService, aber nicht in DampsoftService?

### ✅ **PineconeService (FUNKTIONIERT)**

```typescript
export class PineconeService {  // ← Keine Vererbung
    private async _createIndexAndWait() { ... }
}

// Test funktioniert:
vi.spyOn(service, '_createIndexAndWait' as keyof PineconeService)
```

**Grund:** Bei Klassen **ohne Vererbung** enthält `keyof ClassName` alle eigenen Members, einschließlich privater Methoden zur Compile-Zeit.

### ❌ **DampsoftService (FUNKTIONIERT NICHT)**

```typescript
export class DampsoftService extends PvsBasisService {  // ← Mit Vererbung
    private _getPrax() { ... }
}

// Test schlägt fehl:
vi.spyOn(service, '_getPrax' as keyof DampsoftService)  // ❌ Error
```

**Grund:** Bei **vererbten Klassen** wird `keyof DampsoftService` durch die **abstrakten public Methoden** der Basisklasse dominiert. Private Methoden der abgeleiteten Klasse sind **nicht** im `keyof`-Typ enthalten.

## TypeScript Verhalten Erklärung

### Vererbung beeinflusst `keyof`

```typescript
// PvsBasisService hat abstrakte public Methoden:
abstract class PvsBasisService {
    public abstract synchronizePatients(): Promise<...>
    public abstract getPvsPatients(): Promise<...>
    // ... weitere abstrakte Methoden
}

// DampsoftService erbt diese:
class DampsoftService extends PvsBasisService {
    private _getPrax() { ... }  // ← Diese Methode ist NICHT in keyof enthalten
}

// keyof DampsoftService = "synchronizePatients" | "getPvsPatients" | ...
// aber NICHT "_getPrax"
```

## Lösung: Intersection Type mit expliziter Typisierung

```typescript
// ✅ Korrekte Lösung:
spyOnGetPrax = vi.spyOn(
    // eslint-disable-next-line @typescript-eslint/naming-convention
    service as DampsoftService & { _getPrax: typeof service['_getPrax'] }, 
    '_getPrax'
).mockReturnValue(['PRAX1', 'PRAX2'])
```

### Was passiert hier:

1. **Intersection Type:** `DampsoftService & { _getPrax: ... }` erweitert den Typ
2. **Explizite Typisierung:** `typeof service['_getPrax']` sichert den korrekten Methodentyp
3. **ESLint Disable:** Umgeht Naming-Convention für private Member in Tests
4. **Bracket Notation:** `service['_getPrax']` umgeht die `keyof`-Beschränkung

## Regel für AI-Agents

**WENN** `vi.spyOn()` auf private Methoden fehlschlägt **UND** die Klasse erbt:
1. ✅ Verwende Intersection Type: `service as ClassName & { privateMethod: typeof service['privateMethod'] }`
2. ✅ Füge ESLint-Disable für naming-convention hinzu
3. ❌ Vermeide `as any` Typecasting
4. ❌ Mache private Methoden nicht public

**WENN** die Klasse **nicht erbt:**
- ✅ Standard `keyof ClassName` funktioniert normal

## Technischer Hintergrund

TypeScript's `keyof` Operator verhält sich bei Vererbung restriktiver, da er nur die **öffentlich zugänglichen Members** des kombinierten Typs (Basisklasse + abgeleitete Klasse) berücksichtigt. Private Methoden werden durch die Vererbungshierarchie "versteckt".
    
</details>
















<br><br>
<br><br>





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




