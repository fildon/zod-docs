---
layout: doc-page
title: Basic usage
nav_order: 3
previous:
    title: Installation
    path: ../installation
next:
    title: Defining schemas
    path: ../defining-schemas
---

## Creating a simple string schema

```ts
import { z } from 'zod'

// creating a schema for strings
const stringSchema = z.string()

// parsing
stringSchema.parse( 'Arthur Dent' ) // => 'Arthur Dent'
stringSchema.parse( 42 ) // => throws ZodError

// 'safe' parsing (doesn't throw error if validation fails)
stringSchema.safeParse( 'Arthur Dent' ) // => { success: true; data: 'Arthur Dent' }
stringSchema.safeParse( 42 ) // => { success: false; error: ZodError }
```

## Creating an object schema

```ts
import { z } from 'zod'

const userSchema = z.object( {
    username: z.string(),
} )

type User = z.infer<typeof userSchema> // get the inferred type
// type User = {
//     username: string;
// }

userSchema.parse( { username: 'arthur.dent' } ) // => { username: 'arthur.dent' }
userSchema.parse( { username: 42 } ) // => throws ZodError
```