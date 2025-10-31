# Private Methods – Hashtag (#) privates

Dieser Leitfaden beschreibt eine test-only Strategie für echte ECMAScript-Privates (`#method`/`#field`) mittels Transform-Plugin. Ziel: Whitebox-Zugriff ausschließlich in Unit-Tests – ohne Produktions-Leaks, mit typgesicherter SSOT.

## Überblick
- **Scope:** Nur Unit-Suite; Plugin wird ausschließlich in `vitest.unit.config.ts` registriert.
- **SSOT:** Präfix `TEST_PRIVATES_PREFIX` als Single Source of Truth; Typ `TestPrivatesPrefix`.
- **Ableitung:** Exponierte Keys entstehen aus `${TestPrivatesPrefix}${Extract<keyof PrivateApi, string>}`.
- **Typisierung:** Ambient Module Augmentation mappt PrivateApi → exponierte Keys.

## Setup

### 1) Prefix-SSOT
```ts
// tooling/testing/constants/test-only-transform.constants.ts
export const TEST_PRIVATES_PREFIX = '__test_priv__' as const
export type TestPrivatesPrefix = typeof TEST_PRIVATES_PREFIX
```

### 2) Vitest Unit Config – Plugin registrieren
```ts
// vitest.unit.config.ts (Ausschnitt)
import { defineConfig, mergeConfig } from 'vitest/config'
import baseConfig from './vitest.config'
import { TEST_PRIVATES_PREFIX } from './tooling/testing/constants/test-only-transform.constants'
import { createTestOnlyClassPrivatesRenameTransform } from './tooling/testing/vitest/plugins/rename-class-privates.transform'

export default mergeConfig(baseConfig, defineConfig({
  plugins: [
    createTestOnlyClassPrivatesRenameTransform({ prefix: TEST_PRIVATES_PREFIX })
  ]
}))
```

### 3) Ambient Mapping (type-only)
```ts
// test/types/ambient/<bc>/<module>/<target>.ambient-test.d.ts
import '@services/dampsoft/appointments/core'
import type { DampsoftAppointmentServicePrivateApi } from '@services/dampsoft/appointments/contracts'
import type { TestPrivatesPrefix } from '@tooling/testing/constants/test-only-transform.constants'

type Aug = {
  [K in keyof DampsoftAppointmentServicePrivateApi as `${TestPrivatesPrefix}${string & K}`]:
    DampsoftAppointmentServicePrivateApi[K]
}

declare module '@services/dampsoft/appointments/core' {
  interface DampsoftAppointmentService extends Aug {}
}
```

## Conformance-Checks

### Compile-time (Types)
```ts
import { expectTypeOf, describe, test } from 'vitest'
import type { DampsoftAppointmentService } from '@services/dampsoft/appointments/core'
import type { ReadAppointmentsFn, DampsoftAppointmentServicePrivateApi } from '@services/dampsoft/appointments/contracts'
import type { TestPrivatesPrefix } from '@tooling/testing/constants/test-only-transform.constants'

type K = Extract<keyof DampsoftAppointmentServicePrivateApi, string>
type TransformedKey = `${TestPrivatesPrefix}${K}`

describe('Transform Conformance', () => {
  test('typed transformed member equals PrivateApi contract', () => {
    expectTypeOf<DampsoftAppointmentService[TransformedKey]>()
      .toEqualTypeOf<ReadAppointmentsFn>()
  })
})
```

### Runtime (Unit)
```ts
import { describe, expect, test, vi } from 'vitest'
import { DampsoftAppointmentService } from '@services/dampsoft/appointments/core'
import { TEST_PRIVATES_PREFIX } from '@tooling/testing/constants/test-only-transform.constants'
import type { TestPrivatesPrefix } from '@tooling/testing/constants/test-only-transform.constants'

// Key-Ableitung – strikt typisiert
type TransformedKey = `${TestPrivatesPrefix}readAppointments`

describe('Private #method transform – existence & callability', () => {
  test('exposed method exists and can be spied/called', async () => {
    const service = new DampsoftAppointmentService(/* deps */)
    const key = `${TEST_PRIVATES_PREFIX}readAppointments` as const satisfies TransformedKey

    const spy = vi.spyOn(service as Record<typeof key, unknown>, key as never).mockResolvedValue([])
    const result = await (service as Record<typeof key, () => Promise<unknown[]>>)[key]()

    expect(spy).toHaveBeenCalled()
    expect(Array.isArray(result)).toBe(true)
  })
})
```

## Governance
- Kein Plugin in Base-/Shared-Configs.
- `.d.ts` nur Typen; keine Value-Imports/Side-Effects.
- Produktions-TSConfig schließt `test/**` aus; Test-TSConfig inkludiert Ambient.
- Keine Type-System-Workarounds (`any`, `@ts-ignore`). Schlüssel via `satisfies` validieren.

## Hinweise
- Für Klassen ohne passende Domain-SSOT (`PrivateApi`) zuerst Fn-Aliase und Interface anlegen.
- Für Spies kann auch `prototype` verwendet werden, sofern die Transform-Ausgabe dort landet.
