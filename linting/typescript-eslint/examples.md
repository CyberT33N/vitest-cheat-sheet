# TypeScript-ESLint – Test Examples

## `@typescript-eslint/unbound-method`
**Problem:**  
Directly referencing methods can lead to unexpected `this` behavior.  
If `this` is not used, you can annotate the function with `this: void` or use an arrow function instead.

### Method 1:
In Method 1 erstellen wir eine Wrapper-Funktion (`getPathFn`), die den Aufruf von `app.getPath` übernimmt. Diese Methode stellt sicher, dass:

- **Typprüfung**: Wir prüfen, ob `getPathFn` eine Funktion ist (`expectTypeOf(getPathFn).toBeFunction()`), und validieren den Rückgabewert sowie die Parameter, um sicherzustellen, dass `getPath` den erwarteten Typen liefert.
- **Warum das funktioniert**: Die Wrapper-Funktion garantiert, dass wir den richtigen Typ von `app.getPath` erhalten und gleichzeitig den direkten Aufruf von `app.getPath` vermeiden. Dadurch verhindern wir `this`-Probleme und stellen sicher, dass der Test sauber bleibt.

Diese Methode deckt den Testfall ab, weil wir nicht nur prüfen, ob `getPath` existiert, sondern auch sicherstellen, dass es die richtigen Typen hat und richtig funktioniert.


<details><summary>Click to expand..</summary>

```typescript
// ❌ Error: Directly referencing the method may cause `this` issues
 it('should have correctly typed app.getPath', () => {
     // Check if app.getPath is a function
     expectTypeOf(app.getPath).toBeFunction()
     
     // Check the return type of app.getPath
     expectTypeOf(app.getPath('home')).toBeString()
     
     // Check parameter type - using arrow function to avoid unbound method warning
     const getPathFn = (path: string) => app.getPath(path)
     expectTypeOf(getPathFn).parameter(0).toExtend<string>()
})
```

### ✅ Correct Solution:
```typescript
it('should have correctly typed app.getPath', () => {
// Avoid direct method reference to prevent ESLint warning
const getPathFn = (path: Parameters<typeof app.getPath>[0]): string => app.getPath(path)
      
// Prüfen, ob getPath eine Funktion ist
expectTypeOf(getPathFn).toBeFunction()

// Rückgabewert überprüfen
expectTypeOf(getPathFn('home')).toBeString()

// Parameter-Typ überprüfen
expectTypeOf(getPathFn).parameter(0).toBeString()
});
```

🔹 **Why?**  
- Arrow functions retain the `this` context.  
- Prevents unintended scoping issues.  
- Complies with the ESLint rule `@typescript-eslint/unbound-method`.

</details>
