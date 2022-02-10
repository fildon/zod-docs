---
title: What is Zod?
nav_order: 1
last_modified_date: 2022-02-09
permalink: /
prev:
    title: TODOs
    rel_url: todo
next:
    title: Installation
    rel_url: installation
---

![logo.png](logo.png)
# {{ page.title }}

Zod is a TypeScript-first schema declaration and validation library. I'm using the term 'schema' to broadly refer to any data type, from a simple `string` to a complex nested object.

Zod is designed to be as developer-friendly as possible. The goal is to eliminate duplicative type declarations. With Zod, you declare a validator _once_ and Zod will automatically infer the static TypeScript type. It's easy to compose simpler types into complex data structures.

Some other great aspects:

- Zero dependencies
- Works in Node.js and browsers (including IE 11)
- Tiny: 8kb minified + zipped
- Immutable: methods (i.e. `.optional()`) return a new instance
- Concise, chainable interface
- Functional approach: [parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)
- Works with plain JavaScript too! You don't need to use TypeScript.

{%
    include next-prev-page-links.html
    <!-- prev=page.prev -->
    next=page.next
%}