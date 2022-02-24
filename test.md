---
nav_exclude: true
---

# Test

{% include test.liquid %}


---

## Records

Record schemas are used to validate types such as `{ [k: string]: number }`.

If you want to validate the _values_ of an object against some schema but don't care about the keys, use `Record`.

```ts
const NumberCache = z.record(z.number())

type NumberCache = z.infer<typeof NumberCache>
// => { [k: string]: number }
```

This is particularly useful for storing or caching items by ID.

```ts
const userStore: UserStore = {}

userStore["77d2586b-9e8e-4ecf-8b21-ea7e0530eadd"] = {
  name: "Carlotta",
} // passes

userStore["77d2586b-9e8e-4ecf-8b21-ea7e0530eadd"] = {
  whatever: "Ice cream sundae",
} // TypeError
```

#### A note on numerical keys

You may have expected `z.record()` to accept two arguments, one for the keys and one for the values. After all, TypeScript's built-in Record type does: `Record<KeyType, ValueType>` . Otherwise, how do you represent the TypeScript type `Record<number, any>` in Zod?

As it turns out, TypeScript's behavior surrounding `[k: number]` is a little unintuitive:

```ts
const testMap: { [k: number]: string } = {
  1: "one",
}

for (const key in testMap) {
  console.log(`${key}: ${typeof key}`)
}
// prints: `1: string`
```

As you can see, JavaScript automatically casts all object keys to strings under the hood.

Since Zod is trying to bridge the gap between static and runtime types, it doesn't make sense to provide a way of creating a record schema with numerical keys, since there's no such thing as a numerical key in runtime JavaScript.

## Maps

```ts
const stringNumberMap = z.map(z.string(), z.number())

type StringNumberMap = z.infer<typeof stringNumberMap>
// type StringNumberMap = Map<string, number>
```

## Sets

```ts
const numberSet = z.set(z.number())
type NumberSet = z.infer<typeof numberSet>
// type NumberSet = Set<number>
```

### `.nonempty/.min/.max/.size`

```ts
z.set(z.string()).nonempty() // must contain at least one item
z.set(z.string()).min(5) // must contain 5 or more items
z.set(z.string()).max(5) // must contain 5 or fewer items
z.set(z.string()).size(5) // must contain 5 items exactly
```

## Intersections

<!-- > ⚠️ Intersections are deprecated. If you are trying to merge objects, use the `.merge` method instead. -->

Intersections are useful for creating "logical AND" types. This is useful for intersecting two object types.

```ts
const Person = z.object({
  name: z.string(),
})

const Employee = z.object({
  role: z.string(),
})

const EmployedPerson = z.intersection(Person, Employee)

// equivalent to:
const EmployedPerson = Person.and(Employee)
```

Though in many cases, it is recommended to use `A.merge(B)` to merge two objects. The `.merge` method returns a new `ZodObject` instance, whereas `A.and(B)` returns a less useful `ZodIntersection` instance that lacks common object methods like `pick` and `omit`.

```ts
const a = z.union([z.number(), z.string()])
const b = z.union([z.number(), z.boolean()])
const c = z.intersection(a, b)

type c = z.infer<typeof c> // => number
```

<!-- Intersections in Zod are not smart. Whatever data you pass into `.parse()` gets passed into the two intersected schemas. Because Zod object schemas don't allow any unknown keys by default, there are some unintuitive behavior surrounding intersections of object schemas. -->

<!--

``` ts
const A = z.object({
  a: z.string(),
})

const B = z.object({
  b: z.string(),
})

const AB = z.intersection(A, B)

type Teacher = z.infer<typeof Teacher>
// { id:string name:string }
```  -->

## Recursive types

You can define a recursive schema in Zod, but because of a limitation of TypeScript, their type can't be statically inferred. Instead you'll need to define the type definition manually, and provide it to Zod as a "type hint".

```ts
interface Category {
  name: string
  subcategories: Category[]
}

// cast to z.ZodSchema<Category>
const Category: z.ZodSchema<Category> = z.lazy(() =>
  z.object({
    name: z.string(),
    subcategories: z.array(Category),
  })
)

Category.parse({
  name: "People",
  subcategories: [
    {
      name: "Politicians",
      subcategories: [{ name: "Presidents", subcategories: [] }],
    },
  ],
}) // passes
```

Unfortunately this code is a bit duplicative, since you're declaring the types twice: once in the interface and again in the Zod definition.

<!-- If your schema has lots of primitive fields, there's a way of reducing the amount of duplication:

```ts
// define all the non-recursive stuff here
const BaseCategory = z.object({
  name: z.string(),
  tags: z.array(z.string()),
  itemCount: z.number(),
})

// create an interface that extends the base schema
interface Category extends z.infer<typeof BaseCategory> {
  subcategories: Category[]
}

// merge the base schema with
// a new Zod schema containing relations
const Category: z.ZodSchema<Category> = BaseCategory.merge(
  z.object({
    subcategories: z.lazy(() => z.array(Category)),
  })
)
``` -->

#### JSON type

If you want to validate any JSON value, you can use the snippet below.

```ts
type Literal = boolean | null | number | string
type Json = Literal | { [key: string]: Json } | Json[]
const literalSchema = z.union([z.string(), z.number(), z.boolean(), z.null()])
const jsonSchema: z.ZodSchema<Json> = z.lazy(() =>
  z.union([literalSchema, z.array(jsonSchema), z.record(jsonSchema)])
)

jsonSchema.parse(data)
```

Thanks to [ggoodman](https://github.com/ggoodman) for suggesting this.

#### Cyclical objects

Despite supporting recursive schemas, passing cyclical data into Zod will cause an infinite loop.

## Promises

```ts
const numberPromise = z.promise(z.number())
```

"Parsing" works a little differently with promise schemas. Validation happens in two parts:

1. Zod synchronously checks that the input is an instance of Promise (i.e. an object with `.then` and `.catch` methods.).
2. Zod uses `.then` to attach an additional validation step onto the existing Promise. You'll have to use `.catch` on the returned Promise to handle validation failures.

```ts
numberPromise.parse("tuna")
// ZodError: Non-Promise type: string

numberPromise.parse(Promise.resolve("tuna"))
// => Promise<number>

const test = async () => {
  await numberPromise.parse(Promise.resolve("tuna"))
  // ZodError: Non-number type: string

  await numberPromise.parse(Promise.resolve(3.14))
  // => 3.14
}
```

<!-- #### Non-native promise implementations

When "parsing" a promise, Zod checks that the passed value is an object with `.then` and `.catch` methods — that's it. So you should be able to pass non-native Promises (Bluebird, etc) into `z.promise(...).parse` with no trouble. One gotcha: the return type of the parse function will be a _native_ `Promise` , so if you have downstream logic that uses non-standard Promise methods, this won't work. -->

## Instanceof

You can use `z.instanceof` to check that the input is an instance of a class. This is useful to validate inputs against classes that are exported from third-party libraries.

```ts
class Test {
  name: string
}

const TestSchema = z.instanceof(Test)

const blob: any = "whatever"
TestSchema.parse(new Test()) // passes
TestSchema.parse("blob") // throws
```

## Function schemas

Zod also lets you define "function schemas". This makes it easy to validate the inputs and outputs of a function without intermixing your validation code and "business logic".

You can create a function schema with `z.function(args, returnType)` .

```ts
const myFunction = z.function()

type myFunction = z.infer<typeof myFunction>
// => ()=>unknown
```

**Define inputs and output**

```ts
const myFunction = z
  .function()
  .args(z.string(), z.number()) // accepts an arbitrary number of arguments
  .returns(z.boolean())
type myFunction = z.infer<typeof myFunction>
// => (arg0: string, arg1: number)=>boolean
```

**Extract the input and output schemas**
You can extract the parameters and return type of a function schema.

```ts
myFunction.parameters()
// => ZodTuple<[ZodString, ZodNumber]>

myFunction.returnType()
// => ZodBoolean
```

<!-- `z.function()` accepts two arguments:

* `args: ZodTuple` The first argument is a tuple (created with `z.tuple([...])` and defines the schema of the arguments to your function. If the function doesn't accept arguments, you can pass an empty tuple (`z.tuple([])`).
* `returnType: any Zod schema` The second argument is the function's return type. This can be any Zod schema. -->

> You can use the special `z.void()` option if your function doesn't return anything. This will let Zod properly infer the type of void-returning functions. (Void-returning functions actually return undefined.)

<!--

``` ts
const args = z.tuple([z.string()])

const returnType = z.number()

const myFunction = z.function(args, returnType)
type myFunction = z.infer<typeof myFunction>
// => (arg0: string)=>number
``` -->

Function schemas have an `.implement()` method which accepts a function and returns a new function that automatically validates it's inputs and outputs.

```ts
const trimmedLength = z
  .function()
  .args(z.string()) // accepts an arbitrary number of arguments
  .returns(z.number())
  .implement((x) => {
    // TypeScript knows x is a string!
    return x.trim().length
  })

trimmedLength("sandwich") // => 8
trimmedLength(" asdf ") // => 4
```

If you only care about validating inputs, that's fine:

```ts
const myFunction = z
  .function()
  .args(z.string())
  .implement((arg) => {
    return [arg.length] //
  })
myFunction // (arg: string)=>number[]
```

## Preprocess

Typically Zod operates under a "parse then transform" paradigm. Zod validates the input first, then passes it through a chain of transformation functions. (For more information about transforms, read the [.transform docs](#transform).)

But sometimes you want to apply some transform to the input _before_ parsing happens. A common use case: type coercion. Zod enables this with the `z.preprocess()`.

```ts
const castToString = z.preprocess((val) => String(val), z.string())
```

This returns a `ZodEffects` instance. `ZodEffects` is a wrapper class that contains all logic pertaining to preprocessing, refinements, and transforms.