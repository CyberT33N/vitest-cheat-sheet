# Mocking

## Classes



### Properties




<br><br>
<br><br>

### Public

<details><summary>Click to expand..</summary>

Option1 - `get` :

```typescript
beforeEach(async() => {
    vi.spyOn(context.service, 'patKuerzDBPath', 'get')
                    .mockReturnValue(patKuerzDbPath)
})
```

</details>







<br><br>
<br><br>

### Privat

<details><summary>Click to expand..</summary>

Option1 - `Object.defineProperty` :

```typescript
 beforeEach(async() => {
    // ✅ CLEAN ARCHITECTURE LÖSUNG: Object.defineProperty für readonly private fields
    Object.defineProperty(context.service, '_rechnungDBPath', {
        value: [rechnungDbPath],
        writable: false,
        configurable: true
    })
})
```


</details>





















<br><br>
<br><br>

--- 


<br><br>
<br><br>
