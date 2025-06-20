# Default


## Example 1 - No return value

Wir **KÖNNEN** Module direkt mit **Axios** automodden. Das **PROBLEM** hierbei ist, dass wir kein spezielles **Value** resolven. Das heißt, es wird immer **undefined** in diesen Fällen zurückkommen.
  - Das ist natürlich schön und einfach, wenn wir direkt nur ein Modul mocken **MÜSSEN**, kann aber zu starken Komplikationen führen, wenn Methoden mit dem **RESPONSE** arbeiten. Daher ist diese Variante hier **NUTZLOS**.


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

// AUTO-MOCKING
vi.mock('axios')

describe('UserService', () => {
  it('sollte einen Benutzer abrufen (Auto-Mock)', async () => {
    const user = await userService.getUser(1)
    
    // Will get error because response.data is undefined
  })

  })
})

```











<br><br>
<br><br>




## Example 2 - Return value

- Im Direktvergleich zu **Example 1** oben verwenden wir in den **Tests** ein **Value**, das wir **returnen** würden. Das ist natürlich sinnvoll für **Unit Tests**, weil wir, wenn wir **Packages** aufrufen, mit den **Values** arbeiten **MÜSSEN**.
  - Ein Nachteil dieser Variante ist, dass wir, wenn wir in mehreren Dateien jetzt mit **Axios** arbeiten, jedes Mal das **Neuredundant** definieren **MÜSSEN**. Je nach **Anwendungsfall** würde hier dann eher eine **Mockfactory** **Sinn** machen.

```typescript
import { describe, it, expect, vi } from 'vitest'
import { userService } from '../src/userService.js'
import axios from 'axios'

// AUTO-MOCKING: Verwendet __mocks__/axios.js automatisch
vi.mock('axios')

describe('UserService', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  it.only('PROBLEM: Kann NICHT prüfen ob axios aufgerufen wurde', async () => {
    axios.default.get.mockResolvedValue({ data: { id: 1, name: 'Auto-Mocked User', email: 'auto@mock.com' } })

    await userService.getUser(1)
    
    console.log('axios.get ist:', typeof axios.get) // function
    console.log('axios.get.mock existiert:', !!axios.get.mock) // false

    expect(axios.get).toHaveBeenCalledWith('https://api.example.com/users/1')
    expect(axios.get).toHaveBeenCalledTimes(1)
  })
})

```






