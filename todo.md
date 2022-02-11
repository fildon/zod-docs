# TODO

- [x] todo list could be made (good and easy thing to start)
- [x] Copy docs from readme on github
    - [ ] figure out a way to keep it in sync

- [x] get favicon working
- [ ] set up free domain: https://js.org/
- [ ] move all front matter from docs to pages config file
- [ ] finish ordering nav links
- [ ] finish adding next links to each page
- [ ] figure out how to make next links auto generate

## Separate into sub pages
[pages-with-children](https://pmarsceill.github.io/just-the-docs/docs/navigation-structure/#pages-with-children)
- [ ] Defining schemas
- [ ] Schema methods

## Guides
- [ ] Fetching data from external source and validating the data
- [ ] Custom Error Handling
- [ ] UI Components: https://github.com/colinhacks/zod/discussions/877
    <!-- - [ ] Svelte -->
    <!-- - [ ] React - In progress -->
    <!-- - [ ] Angular -->
    <!-- - [ ] Vue -->
- [ ] Dynamic Schemas. Is this even possible?
    - https://github.com/colinhacks/zod/discussions/908

## Common Mistakes
- [ ] A lot of people expect that the input of a schema should accept anything that be coerced into it, like `z.date` should be able to accept a `date string`, or `z.number` accepting a `string`, etc.
- [ ] Then there is `ZodEffect` which is what you get if you do a `transform` or `refine` call which doesn't allow you to access the methods on the original schema.
- [ ] Better explanation of "Parse, don’t validate"
- [ ] Requiring `strict: true` to get non-optional inference

## Missing Docs
- [ ] CreateOptions argument that allows you to customize the error messages per-schema
- [ ] z.any
- [ ] z.unknown
- [ ] z.never
- [ ] Using Zod with Deno

## Needs more examples
- [ ] z.preprocess
- [ ] Branded types (non-structural "nominal" typing like UUID or URL)

{% comment %}
{% include test.liquid %}
{% endcomment %}