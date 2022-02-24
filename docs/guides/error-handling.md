---
layout: doc-page
title: Error Handling
parent: Guides
nav_order: 1
previous:
    title: Guides
    path: /docs/guides
next:
    title: Ecosystem
    path: /docs/ecosystem
---

This guide explains Zod's internal error handling system, and the various ways you can customize it for your purposes.

## ZodError

All validation errors thrown by Zod are instances of `ZodError`.

```ts
class ZodError extends Error {
  issues: ZodIssue[];
}
```

ZodError is a subclass of `Error`; you can create your own instance easily:

```ts
import * as z from "zod";

const myError = new z.ZodError([]);
```

Each ZodError has an `issues` property that is an array of `ZodIssues`. Each issue documents a problem that occurred during validation.

## ZodIssue

`ZodIssue` is _not_ a class. It is a [discriminated union](https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions).

The link above is the best way to learn about the concept. Discriminated unions are an ideal way to represent a data structures that may be one of many possible variants. You can see all the possible variants defined in the [ZodError.ts source](https://github.com/colinhacks/zod/blob/master/src/ZodError.ts). They are also described in the table below if you prefer.

_Every_ ZodIssue has these fields:

| field     | type                   | details                                                                                           |
| --------- | ---------------------- | ------------------------------------------------------------------------------------------------- |
| `code`    | `z.ZodIssueCode`       | You can access this enum with `z.ZodIssueCode`. A full breakdown of the possible values is below. |
| `path`    | `(string \| number)[]` | e.g, `['addresses', 0, 'line1']`                                                                  |
| `message` | `string`               | e.g. `Invalid type. Expected string, received number.`                                            |

**However** depending on the error code, there may be additional properties as well. Here is a full breakdown of the additional fields by error code:

## ZodIssueCode

| code                             | additional fields                                                                                                                                                                                                                                                                                                                                                    |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ZodIssueCode.invalid_type        | `expected: ZodParsedType` <br> `received: ZodParsedType` <br><br>Jump to [this section](#parsedtype) for a breakdown of the possible values of ZodParsedType.                                                                                                                                                                                                        |
| ZodIssueCode.unrecognized_keys   | `keys: string[]`<br>The list of unrecognized keys<br>                                                                                                                                                                                                                                                                                                                |
| ZodIssueCode.invalid_union       | `unionErrors: ZodError[]` <br> The errors thrown by each element of the union.                                                                                                                                                                                                                                                                                       |
| ZodIssueCode.invalid_enum_value  | `options: string[]` <br> The set of acceptable string values for this enum.                                                                                                                                                                                                                                                                                          |
| ZodIssueCode.invalid_arguments   | `argumentsError: ZodError` <br> This is a special error code only thrown by a wrapped function returned by `ZodFunction.implement()`. The `argumentsError` property is another ZodError containing the validation error details.                                                                                                                                     |
| ZodIssueCode.invalid_return_type | `returnTypeError: ZodError` <br> This is a special error code only thrown by a wrapped function returned by `ZodFunction.implement()`. The `returnTypeError` property is another ZodError containing the validation error details.                                                                                                                                   |
| ZodIssueCode.invalid_date        | _no additional properties_                                                                                                                                                                                                                                                                                                                                           |
| ZodIssueCode.invalid_string      | `validation: "url" \| "email" \| "uuid"`<br> Which built-in string validator failed                                                                                                                                                                                                                                                                                  |
| ZodIssueCode.too_small           | `type: "string" \| "number" \| "array"` <br>The type of the data failing validation<br><br> `minimum: number` <br>The expected length/value.<br><br>`inclusive: boolean`<br>Whether the minimum is included in the range of acceptable values.<br>                                                                                                                   |
| ZodIssueCode.too_big             | `type: "string" \| "number" \| "array"` <br>The type of the data failing validation<br><br> `maximum: number` <br>The expected length/value.<br><br>`inclusive: boolean`<br>Whether the minimum is included in the range of acceptable values.<br>                                                                                                                   |
| ZodIssueCode.not_multiple_of     | `multipleOf: number` <br>The value the number should be a multiple of.<br>                                                                                                                                                                                                                                                                                           |
| ZodIssueCode.custom              | `params: { [k: string]: any }` <br> This is the error code throw by refinements (unless you are using `superRefine` in which case it's possible to throw issues of any `code`). You are able to pass in a `params` object here that is available in your custom error maps (see [ZodErrorMap](#Customizing-errors-with-ZodErrorMap) below for details on error maps) |

<!--
| ZodIssueCode.nonempty_array_is_empty | _no additional properties_                                      |
| ZodIssueCode.invalid_literal_value   | `expected: string \| number \| boolean` <br> The literal value. |
-->

## ZodParsedType

This is an enum used by Zod internally to represent the type of a parsed value. The possible values are:

- `string`
- `nan`
- `number`
- `integer`
- `float`
- `boolean`
- `date`
- `bigint`
- `symbol`
- `function`
- `undefined`
- `null`
- `array`
- `object`
- `unknown`
- `promise`
- `void`
- `never`
- `map`
- `set`

## A demonstrative example

Here's a sample Person schema.

```ts
const person = z.object({
  names: z.array(z.string()).nonempty(), // at least 1 name
  address: z.object({
    line1: z.string(),
    zipCode: z.number().min(10000), // American 5-digit code
  }),
});
```

Let's pass in some improperly formatted data.

```ts
try {
  person.parse({
    names: ["Dave", 12], // 12 is not a string
    address: {
      line1: "123 Maple Ave",
      zipCode: 123, // zip code isn't 5 digits
      extra: "other stuff", // unrecognized key
    },
  });
} catch (err) {
  if (err instanceof z.ZodError) {
    console.log(err.issues);
  }
}
```

Here are the errors that will be printed:

```ts
[
  {
    code: "invalid_type",
    expected: "string",
    received: "number",
    path: ["names", 1],
    message: "Invalid input: expected string, received number",
  },
  {
    code: "unrecognized_keys",
    keys: ["extra"],
    path: ["address"],
    message: "Unrecognized key(s) in object: 'extra'",
  },
  {
    code: "too_small",
    minimum: 10000,
    type: "number",
    inclusive: true,
    path: ["address", "zipCode"],
    message: "Value should be greater than or equal to 10000",
  },
];
```

As you can see three different issues were identified. Every ZodIssue has a `code` property and additional metadata about the validation failure. For instance the `unrecognized_keys` error provides a list of the unrecognized keys detected in the input.

## Customizing errors with ZodErrorMap

You can customize **all** error messages produced by Zod by providing a custom "error map" to Zod, like so:

```ts
import { z } from "zod";

const customErrorMap: z.ZodErrorMap = (issue, ctx) => {
  if (issue.code === z.ZodIssueCode.invalid_type) {
    if (issue.expected === "string") {
      return { message: "bad type!" };
    }
  }
  if (issue.code === z.ZodIssueCode.custom) {
    return { message: `less-than-${(issue.params || {}).minimum}` };
  }
  return { message: ctx.defaultError };
};

z.setErrorMap(customErrorMap);
```

`ZodErrorMap` is a special function. It accepts two arguments: `issue` and `ctx`. The return type is `{ message: string }`. Essentially the error map accepts some information about the validation that is failing and returns an appropriate error message.

- `issue: Omit<ZodIssue, "message">`

  As mentioned above, ZodIssue is a discriminated union.

- `ctx: { defaultError: string; data: any }`

  - `ctx.default` is the error message generated by the default error map. If you only want to override the message for a single type of error, you can do that. Just return `defaultError` for everything

  - `ctx.data` contains the data that was passed into `.parse`. You can use this to customize the error message.

As in the example, you can modify certain error messages and simply fall back to `ctx.defaultError` otherwise.

## Error map priority

A custom error maps doesn't need to produce an error message for every kind of issue in Zod. Instead, your error map can override certain errors and return `ctx.defaultError` for everything else.

But how is the value of `ctx.defaultError` determined?

Error messages in Zod are generated by passing metadata about a validation issue through a chain of error maps. Error maps with higher priority override messages generated by maps with lower priority.

The lowest priority map is the `defaultErrorMap`, which defined in [`src/ZodError.ts`](https://github.com/colinhacks/zod/blob/master/src/ZodError.ts). This produces the default error message for all issues in Zod.

### Global error map

This message is then passed as `ctx.defaultError` into `overrideErrorMap`. This is a global error map you can set with `z.setErrorMap`:

```ts
const myErrorMap: z.ZodErrorMap = /* ... */;
z.setErrorMap(errorMap);
```

### Schema-bound error map

The `overrideErrorMap` message is then passed as `ctx.defaultError` into any schema-bound error maps. Every schema can be associated with an error map.

```ts
z.string({ errorMap: myErrorMap });

// this creates an error map under the hood
z.string({
  invalid_type_error: "Invalid name",
  required_error: "Name is required",
});
```

### Contextual error map

Finally, you can pass an error map as a parameter to any `parse` method. This error map, if provided, has highest priority.

```ts
z.string().parse("adsf", { errorMap: myErrorMap });
```

## A working example

Let's look at a practical example of of customized error map:

```ts
import * as z from "zod";

const errorMap: z.ZodErrorMap = (error, ctx) => {
  /*
  This is where you override the various error codes
  */
  switch (error.code) {
    case z.ZodIssueCode.invalid_type:
      if (error.expected === "string") {
        return { message: `This ain't a string!` };
      }
      break;
    case z.ZodIssueCode.custom:
      // produce a custom message using error.params
      // error.params won't be set unless you passed
      // a `params` arguments into a custom validator
      const params = error.params || {};
      if (params.myField) {
        return { message: `Bad input: ${params.myField}` };
      }
      break;
  }

  // fall back to default message!
  return { message: ctx.defaultError };
};

z.string().parse(12, { errorMap });

/* throws: 
  ZodError {
    errors: [{
      code: "invalid_type",
      path: [],
      message: "This ain't a string!",
      expected: "string",
      received: "number",
    }]
  }
*/
```

## Error handling for forms

If you're using Zod to validate the inputs from a web form, there is a convenient way to "flatten" a ZodError to a format that can be easily displayed to the end user.

Consider this example of a simple signup form:

```ts
const FormData = z
  .object({
    email: z.string().email(),
    password: z.string().min(10),
    confirm: z.string().min(10),
  })
  .refine(obj => obj.password === obj.confirm, {
    message: 'Passwords do not match',
    path: ['confirm'], // this value is concatenated to the end of the actual path of the error
  });
}
```

Now lets pass in some invalid data:

```ts
FormData.parse({
  email: "not an email",
  password: "tooshort",
  confirm: "nomatch",
});
```

This will throw a ZodError with four issues:

```ts
console.log(err.issues);
/*
  [
    { code: 'invalid_string', validation: 'email', path: ['email'], message: 'Invalid email' },
    {
      code: 'too_small',
      minimum: 10,
      type: 'string',
      inclusive: true,
      path: ['password'],
      message: 'Should be at least 10 characters',
    },
    {
      code: 'too_small',
      minimum: 10,
      type: 'string',
      inclusive: true,
      path: ['confirm'],
      message: 'Should be at least 10 characters',
    },
    { code: 'custom', message: 'Passwords do not match', path: ['confirm'] },
  ]; 
  */
```

But using the `flatten()` method, we can make those errors much easier to work with:

```ts
console.log(err.flatten());
/*
  {
  formErrors: [],
  fieldErrors: {
    email: ['Invalid email'],
    password: ['Should be at least 10 characters'],
    confirm: ['Should be at least 10 characters', 'Passwords do not match'],
  },
}
*/
```

Additionally, you can pass a mapping function of `(issue: ZodIssue) => U` to `flatten()`, which is applied to the during the transformation of `ZodIssue`s.
This can be particularly useful when integrating Zod with form validation, as it allows you to pass back whatever `ZodIssue` specific context you might need.

```ts
err.flatten( (i: ZodIssue) => {
  message: i.message,
  errorCode: i.code
});
/*
  {
  formErrors: [],
  fieldErrors: {
    email: [{ message: 'Invalid email', errorCode: 'invalid_string' }],
    password: [{ message: 'Should be at least 10 characters', errorCode: 'too_small' }],
    confirm: [{ message: 'Should be at least 10 characters', errorCode: 'too_small' }, { message: 'Passwords do not match', errorCode: 'custom' }],
  },
}
*/
```

- `fieldErrors` is an object. The keys are the field(s) that threw the error. The values are an array of error strings that can be easily presented in the interface.
- `formErrors: string[]` is an array of errors that occurred on the "root" of the object schema. For instance if you called `FormData.parse(null)`, `formErrors` would be:
  ```ts
  ["Invalid input: expected object, received null"];
  ```

## Type-safety with `safeParse`.

If you're using `safeParse`, and need more type-safety around error results, you can use `z.inferFlattenedErrors` to infer the result of error result called with `flatten()`, based on your schema.

```ts
type FormDataErrors = z.inferFlattenedErrors<typeof FormData>;

/*
  formErrors: string[],
  fieldErrors: {
    email?: string[],
    password?: string[],
    confirm?: string[]
  } 
*/
```

By default all error types are assumed to be `string`. If you're using a mapping function to transform `ZodIssue`s, you can provide the error type to `z.inferFlattenedErrors`.

```ts
type FormDataErrors = z.inferFlattenedErrors<typeof FormData, { message: string, errorCode: string }>;

/*
  formErrors: { message: string, errorCode: string }[],
  fieldErrors: {
    email?: { message: string, errorCode: string }[],
    password?: { message: string, errorCode: string }[],
    confirm?: { message: string, errorCode: string }[]
  } 
*/

const result = FormData.safeParse({
  email: "not email",
  password: "tooshort",
  confirm: "nomatch",
})

if (!result.success) {
  let bad: FormDataErrors = err.flatten(); // Type error: Type 'string' is not assignable to type '{ message: string; }'.
  let good: FormDataErrors = err.flatten(i => ({
    message: i.message,
    errorCode: i.code
  }))
}
```

Additionally, you can use `z.inferFormErrors` as a convienience for `z.inferFlattenedErrors<T, string>` in combination with `formErrors`.

```ts
type FormDataErrors = z.inferFormErrors<typeof FormData, { message: string, errorCode: string }>;

let formErrors: FormDataErrors = result.error.formErrors;
```
