---
layout: doc-page
title: Literals
parent: Defining schemas
nav_order: 2
previous:
    title: Primitives
    rel_url: ../primitives
next:
    title: Strings
    rel_url: ../strings
---

```ts
const tuna = z.literal("tuna");
const twelve = z.literal(12);
const tru = z.literal(true);

// retrieve literal value
tuna.value; // "tuna"
```

> Currently there is no support for Date or bigint literals in Zod. If you have a use case for this feature, please file an issue.