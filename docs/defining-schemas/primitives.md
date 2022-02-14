---
layout: doc-page
title: Primitives
parent: Defining schemas
nav_order: 1
previous:
    title: Defining schemas
    rel_url: ../../defining-schemas
next:
    title: Literals
    rel_url: ../literals
---

```ts
// primitive values
z.string();
z.number();
z.bigint();
z.boolean();
z.date();

// empty types
z.undefined();
z.null();
z.void(); // accepts undefined

// catch-all types
// allows any value
z.any();
z.unknown();

// never type
// allows no values
z.never();
```