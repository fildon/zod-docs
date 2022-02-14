---
layout: doc-page
title: Numbers
parent: Defining schemas
nav_order: 4
previous:
    title: Strings
    rel_url: ../strings
next:
    title: Booleans
    rel_url: ../booleans
---

## Modifiers

```ts
z.number().gt(5);
z.number().gte(5); // alias .min(5)
z.number().lt(5);
z.number().lte(5); // alias .max(5)

z.number().int(); // value must be an integer

z.number().positive(); //     > 0
z.number().nonnegative(); //  >= 0
z.number().negative(); //     < 0
z.number().nonpositive(); //  <= 0

z.number().multipleOf(5); // Evenly divisible by 5. Alias .step(5)
```

## Custom error messages

You can customize certain error messages when creating a schema.

```ts
const age = z.number({
  required_error: "Age is required",
  invalid_type_error: "Age must be a number",
});
```

## Modifiers with custom error messages
When using modifiers, you can pass in a second argument to provide a custom error message.

```ts
z.number().lte(5, { message: "this👏is👏too👏big" });
```