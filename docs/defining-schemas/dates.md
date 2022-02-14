---
layout: doc-page
title: Dates
parent: Defining schemas
nav_order: 6
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
const dateSchema = z.date({
  required_error: "dateSchema is required",
  invalid_type_error: "dateSchema must be a Date",
});
```

## [Coercion](https://developer.mozilla.org/en-US/docs/Glossary/Type_coercion)
z.date() accepts a date, NOT a date string
```ts
z.date().safeParse( new Date() ) // success: true
z.date().safeParse( '2022-01-12T00:00:00.000Z' ) // success: false
```

To coerce any input into a Date, you can use [preprocess]() // TODO add link
```ts
const dateSchema = z.preprocess(
    input => new Date( input as any ),
    z.date()
)
type DateSchema = z.infer<typeof dateSchema>
// type DateSchema = Date

dateSchema.safeParse( new Date() ) // success: true
dateSchema.safeParse( '2022-01-12T00:00:00.000Z' ) // success: true
dateSchema.safeParse( 1641945600000 ) // success: true
```
⚠️ Be careful, this might allow unintended values
```ts
dateSchema.safeParse( null ) // success: true
```

To make sure you are only allowing the values you intend, check the type of `input` before you return the new value

```ts
const dateSchema = z.preprocess(
    input => {
        if ( input instanceof Date ) return input
        if ( typeof input == 'string' || typeof input == 'number' )
            return new Date( input )
    },
    z.date()
)
type DateSchema = z.infer<typeof dateSchema>
// type DateSchema = Date

dateSchema.safeParse( new Date() ) // success: true
dateSchema.safeParse( '2022-01-12T00:00:00.000Z' ) // success: true
dateSchema.safeParse( 1641945600000 ) // success: true
dateSchema.safeParse( null ) // success: false
```