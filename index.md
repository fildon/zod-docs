# Basic usage

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
<!-- 
---
- [Next: Defining schemas](defining-schemas/README.md)
- [Table of Contents](README.md)
-->



<!-- https://pmarsceill.github.io/just-the-docs/ -->
<!-- https://github.com/pmarsceill/just-the-docs/blob/master/_config.yml -->

<!-- ---
title: Basic usage
nav_order: 1
--- -->