# Usage without vi.fn access


```typescript
// Auto-Mock für axios (OHNE Vitest Mock-Funktionen)
export default {
  get: () => Promise.resolve({ 
    data: { id: 1, name: 'Auto-Mocked User', email: 'auto@mock.com' } 
  }),
  post: () => Promise.resolve({ 
    data: { id: 2, name: 'Created User', email: 'created@mock.com' } 
  }),
  delete: () => Promise.resolve({ status: 200 })
}

```

```typescript
import axios from 'axios'

export class UserService {
  async getUser(id) {
    const response = await axios.get(`https://api.example.com/users/${id}`)
    return response.data
  }

  async createUser(userData) {
    const response = await axios.post('https://api.example.com/users', userData)
    return response.data
  }

  async deleteUser(id) {
    await axios.delete(`https://api.example.com/users/${id}`)
    return { success: true }
  }
}

export const userService = new UserService()

```

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






