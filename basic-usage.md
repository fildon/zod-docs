---
title: Basic usage
nav_order: 3
last_modified_date: 2022-02-09
---

# {{ page.title }}

Creating a simple string schema

```ts
import { z } from "zod";

// creating a schema for strings
const mySchema = z.string();

// parsing
mySchema.parse("tuna"); // => "tuna"
mySchema.parse(12); // => throws ZodError

// "safe" parsing (doesn't throw error if validation fails)
mySchema.safeParse("tuna"); // => { success: true; data: "tuna" }
mySchema.safeParse(12); // => { success: false; error: ZodError }
```

Creating an object schema

```ts
import { z } from "zod";

const User = z.object({
    username: z.string(),
});

User.parse({ username: "Ludwig" });

// extract the inferred type
type User = z.infer<typeof User>;
// { username: string }
```

{%
    include next-page-link.html
    text='Next: Defining schemas'
    link='../defining-schemas'
%}