---
layout: doc-page
title: Booleans
parent: Defining schemas
nav_order: 5
previous:
    title: Numbers
    rel_url: ../numbers
next:
    title: Dates
    rel_url: ../dates
---

## Custom error messages

You can customize certain error messages when creating a schema.

```ts
const isActive = z.boolean({
  required_error: "isActive is required",
  invalid_type_error: "isActive must be a boolean",
});
```