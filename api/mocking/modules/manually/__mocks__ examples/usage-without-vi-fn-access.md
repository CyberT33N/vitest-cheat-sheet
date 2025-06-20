# Usage without vi.fn access

```typescript
import { describe, it, expect, vi } from 'vitest'
import { userService } from '../src/userService.js'
import axios from 'axios'

// AUTO-MOCKING: Verwendet __mocks__/axios.js automatisch
vi.mock('axios')

describe('UserService - Auto-Mock (__mocks__ Variante)', () => {
  it('sollte einen Benutzer abrufen (Auto-Mock)', async () => {
    const user = await userService.getUser(1)
    
    // ✅ Das funktioniert - wir bekommen die gemockten Daten
    expect(user).toEqual({
      id: 1,
      name: 'Auto-Mocked User',
      email: 'auto@mock.com'
    })
  })

  it.only('PROBLEM: Kann NICHT prüfen ob axios aufgerufen wurde', async () => {
    await userService.getUser(1)
    
    // ❌ DAS FUNKTIONIERT NICHT!
    // axios ist kein Vitest Mock, sondern unser statischer Mock aus __mocks__
    // expect(axios.get).toHaveBeenCalledWith('https://api.example.com/users/1')
    
    console.log('axios.get ist:', typeof axios.get) // function
    console.log('axios.get.mock existiert:', !!axios.get.mock) // false
    
    // ❌ Diese Assertion wird fehlschlagen:
    try {
      expect(axios.get).toHaveBeenCalledWith('https://api.example.com/users/1')
    } catch (error) {
      console.log('Fehler (erwartet):', error.message)
      // Zeigt: "expect.toHaveBeenCalledWith is not supported"
    }
  })
})

```






