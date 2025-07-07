# Mocking

## Classes



### Properties



### Private

<details><summary>Click to expand..</summary>

Option1 - `ClassName.prototype` :

```typescript
beforeEach(async() => {
    vi.spyOn(context.service, '_rechnungDBPath', 'get')
        .mockReturnValue(patKuerzDbPath)
})
```

</details>





<br><br>
<br><br>

--- 


<br><br>
<br><br>
