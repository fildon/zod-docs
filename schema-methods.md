---
title: Schema methods
nav_order: 5
last_modified_date: 2022-02-09
---

# {{ page.title }}

All Zod schemas contain these methods.

- [.parse](#parse)
- [.parseAsync](#parseasync)
- [.safeParse](#safeparse)
- [.safeParseAsync](#safeparseasync)
- [.refine](#refine)
- [.superRefine](#superRefine)
- [.transform](#transform)
- [.default](#default)
- [.optional](#optional)
- [.nullable](#nullable)
- [.nullish](#nullish)
- [.array](#array)
- [.promise](#promise)
- [.or](#or)
- [.and](#and)

---

### `.parse`

`.parse(data:unknown): T`

Given any Zod schema, you can call its `.parse` method to check `data` is valid. If it is, a value is returned with full type information! Otherwise, an error is thrown.

> IMPORTANT: In Zod 2 and Zod 1.11+, the value returned by `.parse` is a _deep clone_ of the variable you passed in. This was also the case in zod@1.4 and earlier.

```ts
const stringSchema = z.string();
stringSchema.parse("fish"); // => returns "fish"
stringSchema.parse(12); // throws Error('Non-string type: number');
```

### `.parseAsync`

`.parseAsync(data:unknown): Promise<T>`

If you use asynchronous [refinements](#refine) or [transforms](#transform) (more on those later), you'll need to use `.parseAsync`

```ts
const stringSchema = z.string().refine(async (val) => val.length > 20);
const value = await stringSchema.parseAsync("hello"); // => hello
```

### `.safeParse`

`.safeParse(data:unknown): { success: true; data: T; } | { success: false; error: ZodError; }`

If you don't want Zod to throw errors when validation fails, use `.safeParse`. This method returns an object containing either the successfully parsed data or a ZodError instance containing detailed information about the validation problems.

```ts
stringSchema.safeParse(12);
// => { success: false; error: ZodError }

stringSchema.safeParse("billie");
// => { success: true; data: 'billie' }
```

The result is a _discriminated union_ so you can handle errors very conveniently:

```ts
const result = stringSchema.safeParse("billie");
if (!result.success) {
  // handle error then return
  result.error;
} else {
  // do something
  result.data;
}
```

### `.safeParseAsync`

> Alias: `.spa`

An asynchronous version of `safeParse`.

```ts
await stringSchema.safeParseAsync("billie");
```

For convenience, this has been aliased to `.spa`:

```ts
await stringSchema.spa("billie");
```

### `.refine`

`.refine(validator: (data:T)=>any, params?: RefineParams)`

Zod lets you provide custom validation logic via _refinements_. (For advanced features like creating multiple issues and customizing error codes, see [`.superRefine`](#superrefine).)

Zod was designed to mirror TypeScript as closely as possible. But there are many so-called "refinement types" you may wish to check for that can't be represented in TypeScript's type system. For instance: checking that a number is an integer or that a string is a valid email address.

For example, you can define a custom validation check on _any_ Zod schema with `.refine` :

```ts
const myString = z.string().refine((val) => val.length <= 255, {
  message: "String can't be more than 255 characters",
});
```

> ⚠️ Refinement functions should not throw. Instead they should return a falsy value to signal failure.

#### Arguments

As you can see, `.refine` takes two arguments.

1. The first is the validation function. This function takes one input (of type `T` — the inferred type of the schema) and returns `any`. Any truthy value will pass validation. (Prior to zod@1.6.2 the validation function had to return a boolean.)
2. The second argument accepts some options. You can use this to customize certain error-handling behavior:

```ts
type RefineParams = {
  // override error message
  message?: string;

  // appended to error path
  path?: (string | number)[];

  // params object you can use to customize message
  // in error map
  params?: object;
};
```

For advanced cases, the second argument can also be a function that returns `RefineParams`/

```ts
z.string().refine(
  (val) => val.length > 10,
  (val) => ({ message: `${val} is not more than 10 characters` })
);
```

#### Customize error path

```ts
const passwordForm = z
  .object({
    password: z.string(),
    confirm: z.string(),
  })
  .refine((data) => data.password === data.confirm, {
    message: "Passwords don't match",
    path: ["confirm"], // path of error
  })
  .parse({ password: "asdf", confirm: "qwer" });
```

Because you provided a `path` parameter, the resulting error will be:

```ts
ZodError {
  issues: [{
    "code": "custom",
    "path": [ "confirm" ],
    "message": "Passwords don't match"
  }]
}
```

#### Asynchronous refinements

Refinements can also be async:

```ts
const userId = z.string().refine(async (id) => {
  // verify that ID exists in database
  return true;
});
```

> ⚠️If you use async refinements, you must use the `.parseAsync` method to parse data! Otherwise Zod will throw an error.

#### Relationship to transforms

Transforms and refinements can be interleaved:

```ts
z.string()
  .transform((val) => val.length)
  .refine((val) => val > 25);
```

<!-- Note that the `path` is set to `["confirm"]` , so you can easily display this error underneath the "Confirm password" textbox.


```ts
const allForms = z.object({ passwordForm }).parse({
  passwordForm: {
    password: "asdf",
    confirm: "qwer",
  },
});
```

would result in

```

ZodError {
  issues: [{
    "code": "custom",
    "path": [ "passwordForm", "confirm" ],
    "message": "Passwords don't match"
  }]
}
``` -->

### `.superRefine`

The `.refine` method is actually syntactic sugar atop a more versatile (and verbose) method called `superRefine`. Here's an example:

```ts
const Strings = z.array(z.string()).superRefine((val, ctx) => {
  if (val.length > 3) {
    ctx.addIssue({
      code: z.ZodIssueCode.too_big,
      maximum: 3,
      type: "array",
      inclusive: true,
      message: "Too many items 😡",
    });
  }

  if (val.length !== new Set(val).size) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: `No duplicated allowed.`,
    });
  }
});
```

You can add as many issues as you like. If `ctx.addIssue` is NOT called during the execution of the function, validation passes.

Normally refinements always create issues with a `ZodIssueCode.custom` error code, but with `superRefine` you can create any issue of any code. Each issue code is described in detail in the Error Handling guide: [ERROR_HANDLING.md](ERROR_HANDLING.md).

#### Abort early

By default, parsing will continue even after a refinement check fails. For instance, if you chain together multiple refinements, they will all be executed. However, it may be desirable to _abort early_ to prevent later refinements from being executed. To achieve this, pass the `fatal` flag to `ctx.addIssue`:

```ts
const Strings = z
  .number()
  .superRefine((val, ctx) => {
    if (val < 10) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "foo",
        fatal: true,
      });
    }
  })
  .superRefine((val, ctx) => {
    if (val !== " ") {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "bar",
      });
    }
  });
```

### `.transform`

To transform data after parsing, use the `transform` method.

```ts
const stringToNumber = z.string().transform((val) => myString.length);
stringToNumber.parse("string"); // => 6
```

> ⚠️ Transform functions must not throw. Make sure to use refinements before the transform to make sure the input can be parsed by the transform.

#### Chaining order

Note that `stringToNumber` above is an instance of the `ZodEffects` subclass. It is NOT an instance of `ZodString`. If you want to use the built-in methods of `ZodString` (e.g. `.email()`) you must apply those methods _before_ any transforms.

```ts
const emailToDomain = z
  .string()
  .email()
  .transform((val) => val.split("@")[1]);

emailToDomain.parse("colinhacks@example.com"); // => example.com
```

#### Relationship to refinements

Transforms and refinements can be interleaved:

```ts
z.string()
  .transform((val) => val.length)
  .refine((val) => val > 25);
```

#### Async transforms

Transforms can also be async.

```ts
const IdToUser = z
  .string()
  .uuid()
  .transform(async (id) => {
    return await getUserById(id);
  });
```

> ⚠️ If your schema contains asynchronous transforms, you must use .parseAsync() or .safeParseAsync() to parse data. Otherwise Zod will throw an error.

### `.default`

You can use transforms to implement the concept of "default values" in Zod.

```ts
const stringWithDefault = z.string().default("tuna");

stringWithDefault.parse(undefined); // => "tuna"
```

Optionally, you can pass a function into `.default` that will be re-executed whenever a default value needs to be generated:

```ts
const numberWithRandomDefault = z.number().default(Math.random);

numberWithRandomDefault.parse(undefined); // => 0.4413456736055323
numberWithRandomDefault.parse(undefined); // => 0.1871840107401901
numberWithRandomDefault.parse(undefined); // => 0.7223408162401552
```

### `.optional`

A convenience method that returns an optional version of a schema.

```ts
const optionalString = z.string().optional(); // string | undefined

// equivalent to
z.optional(z.string());
```

### `.nullable`

A convenience method that returns an nullable version of a schema.

```ts
const nullableString = z.string().nullable(); // string | null

// equivalent to
z.nullable(z.string());
```

### `.nullish`

A convenience method that returns a "nullish" version of a schema. Nullish schemas will accept both `undefined` and `null`. Read more about the concept of "nullish" [here](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#nullish-coalescing).

```ts
const nullishString = z.string().nullish(); // string | null | undefined

// equivalent to
z.string().optional().nullable();
```

### `.array`

A convenience method that returns an array schema for the given type:

```ts
const nullableString = z.string().array(); // string[]

// equivalent to
z.array(z.string());
```

### `.promise`

A convenience method for promise types:

```ts
const stringPromise = z.string().promise(); // Promise<string>

// equivalent to
z.promise(z.string());
```

### `.or`

A convenience method for union types.

```ts
z.string().or(z.number()); // string | number

// equivalent to
z.union([z.string(), z.number()]);
```

### `.and`

A convenience method for creating intersection types.

```ts
z.object({ name: z.string() }).and(z.object({ age: z.number() })); // { name: string } & { age: number }

// equivalent to
z.intersection(z.object({ name: z.string() }), z.object({ age: z.number() }));
```

{%
    include next-page-link.html
    text='Next: Type inference'
    link='../type-inference'
%}