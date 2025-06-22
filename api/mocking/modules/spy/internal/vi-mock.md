
# Spy Module
- Mockt das Modul, behält aber Implementierung bei

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { userService } from '../src/userService.js'

vi.mock('axios', { spy: true })

const axios = await import('axios')

describe('UserService - Spy Variante (spy: true)', () => {
  aftzerEach(() => {
    vi.clearAllMocks()
  })

  it('sollte axios Aufrufe überwachen können', async () => {
    // Mit spy: true können wir sowohl die originale Implementierung 
    // als auch die Mock-Überwachung nutzen
    
    // Setup eines Rückgabewerts
    const mockData = { id: 1, name: 'Spy User', email: 'spy@test.com' }
    axios.default.get.mockResolvedValue({ data: mockData })
    
    const user = await userService.getUser(1)
    
    expect(user).toEqual(mockData)
    
    // ✅ Spy-Funktionalität: Aufrufe können überwacht werden
    expect(axios.default.get).toHaveBeenCalledWith('https://api.example.com/users/1')
    expect(axios.default.get).toHaveBeenCalledTimes(1)
  })
})

```