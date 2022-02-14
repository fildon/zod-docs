---
layout: doc-page
title: Strings
parent: Defining schemas
nav_order: 3
previous:
    title: Literals
    rel_url: ../literals
next:
    title: Numbers
    rel_url: ../numbers
---

## Modifiers

```ts
z.string().max(5);
z.string().min(5);
z.string().length(5);
z.string().email();
z.string().url();
z.string().uuid();
z.string().cuid();
z.string().regex(regex);

// deprecated, equivalent to .min(1)
z.string().nonempty();

// optional custom error message
z.string().nonempty({ message: "Can't be empty" });
```

<!--
Check out [validator.js](https://github.com/validatorjs/validator.js) for a bunch of other useful string validation functions.
-->

## Custom error messages

You can customize certain error messages when creating a schema.

```ts
const name = z.string({
  required_error: "Name is required",
  invalid_type_error: "Name must be a string",
});
```

## Modifiers with custom error messages
When using modifiers, you can pass in a second argument to provide a custom error message.

```ts
z.string().min(5, { message: "Must be 5 or more characters long" });
z.string().max(5, { message: "Must be 5 or fewer characters long" });
z.string().length(5, { message: "Must be exactly 5 characters long" });
z.string().email({ message: "Invalid email address" });
z.string().url({ message: "Invalid url" });
z.string().uuid({ message: "Invalid UUID" });
```