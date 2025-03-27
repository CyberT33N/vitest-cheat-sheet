# expectTypeOf

Die `expectTypeOf`-Funktion wird verwendet, um Typen in TypeScript zu testen. Diese API ermöglicht es, die Typen von Funktionsparametern, Rückgabewerten und mehr zu überprüfen.

Offizielle Dokumentation: [https://vitest.dev/api/expect-typeof](https://vitest.dev/api/expect-typeof)

## Private Klassen-Konstruktoren

Es gibt keine direkte Möglichkeit, einen privaten Konstruktor auf Parametertypen zu testen. Eine Lösung ist, eine spezielle Testmethode zu erstellen:

```typescript
class Foo {
    private constructor(private x: number) {}

    /** @internal */
    static createForTesting(x: number) {
        return new Foo(x);
    }
}

const instance = Foo.createForTesting(5);
```

Wenn Sie jedoch `getInstance` als Single-Method verwenden und dies der Grund ist, warum Ihr Konstruktor privat ist, können Sie diese Methode testen.

## toEqualTypeOf

Überprüft, ob ein Typ einem anderen Typ entspricht:

```typescript
expectTypeOf({ a: 1 }).toEqualTypeOf<{ a: number }>()
expectTypeOf({ a: 1 }).toEqualTypeOf({ a: 1 })
expectTypeOf({ a: 1 }).toEqualTypeOf({ a: 2 })
expectTypeOf({ a: 1, b: 1 }).not.toEqualTypeOf<{ a: number }>()
```

## Generics

Sie können auch Generic-Typen überprüfen:

```typescript
expectTypeOf(modelManager.createModel<TMongooseSchema>).parameter(0).toEqualTypeOf<IModelCore<TMongooseSchema>>()

/**
 * Creates a Mongoose model based on the given name, schema, and database name.
 * @template TMongooseSchema - The type of the mongoose schema.
 * @param modelDetails - An object containing the model's details.
 * @returns A promise that resolves to the created Mongoose Model instance.
 */
public async createModel<TMongooseSchema>({
    modelName,
    schema,
    dbName
}: IModelCore<TMongooseSchema>): Promise< mongoose.Model<TMongooseSchema> > {
    const mongooseUtils = await MongooseUtils.getInstance(dbName)
    const Model = await mongooseUtils.createModel<TMongooseSchema>(schema, modelName)
    
    // Ensure indexes are created for the model
    await Model.createIndexes()
    return Model
}
```

## parameter

Extracts a parameter type from a function.

Offizielle Dokumentation: [https://vitest.dev/api/expect-typeof.html#parameter](https://vitest.dev/api/expect-typeof.html#parameter)

```typescript
expectTypeOf(modelManager['globModels']).parameter(0).toBeString()
expectTypeOf(modelManager.createModel<TMongooseSchema>).parameter(0).toEqualTypeOf<IModelCore<TMongooseSchema>>()
```

**Hinweis**: Dies funktioniert nicht mit statischen Methoden in Klassen. Verwenden Sie stattdessen `.toBeCallableWith()`.

## toBeCallableWith

Überprüft, ob eine Funktion mit bestimmten Parametern aufgerufen werden kann.

Offizielle Dokumentation: [https://vitest.dev/api/expect-typeof.html#tobecallablewith](https://vitest.dev/api/expect-typeof.html#tobecallablewith)

```typescript
expectTypeOf(ModelUtils.createMemoryModel).toBeCallableWith(modelCoreDetails)
```

## returns

Extrahiert den Rückgabewert eines Funktionstyps.

```typescript
import { expectTypeOf } from 'vitest'

expectTypeOf(() => {}).returns.toBeVoid()
expectTypeOf((a: number) => [a, a]).returns.toEqualTypeOf([1, 2])
expectTypeOf(modelManager['getModels']).returns.toEqualTypeOf<IModel<mongoose.SchemaDefinition<{}>>[]>()

// Bei Bedarf können Sie ein Promise selbst aufrufen und mit returns überprüfen,
// aber es wird empfohlen, resolve zu verwenden
expectTypeOf(ModelManager.getInstance).returns.toEqualTypeOf(Promise.resolve(modelManager))
```

## resolve

Extrahiert den aufgelösten Wert eines Promise, sodass Sie weitere Assertions daran durchführen können.

```typescript
import { expectTypeOf } from 'vitest'

async function asyncFunc() {
  return 123
}

expectTypeOf(asyncFunc).returns.resolves.toBeNumber()
expectTypeOf(Promise.resolve('string')).resolves.toBeString()

// WEITERE BEISPIELE
expectTypeOf(ModelManager.getInstance).returns.resolves.toEqualTypeOf<ModelManager>()

// Private Klassenmethoden
expectTypeOf(modelManager['globModels']).returns.resolves.toEqualTypeOf<IModel<mongoose.SchemaDefinition<{}>>[]>()
expectTypeOf(modelManager['init']).returns.resolves.toBeVoid()
```

## instance

Ermöglicht den Zugriff auf Matcher, die an einer Instanz der bereitgestellten Klasse durchgeführt werden können.

Offizielle Dokumentation: [https://vitest.dev/api/expect-typeof#instance](https://vitest.dev/api/expect-typeof#instance)

```typescript
import { expectTypeOf } from 'vitest'

expectTypeOf(Date).instance.toHaveProperty('toISOString')
``` 