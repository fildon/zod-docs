---
title: Installation
nav_order: 2
last_modified_date: 2022-02-09
prev:
    title: What is Zod?
    rel_url: ../
next:
    title: Basic usage
    rel_url: ../basic-usage
---

# {{ page.title }}

## Node
To install Zod v3:

```sh
npm install zod
```

---

## TypeScript
⚠️ IMPORTANT: You must enable `strict` mode in your `tsconfig.json`. This is a best practice for all TypeScript projects.

```ts
// tsconfig.json
{
    "compilerOptions": {
        "strict": true
    }
}
```

| Zod Version | Required TypeScript Version |
| :--: | :--: |
| 3.x | 4.1+ |
| 2.x | 3.7+ |
| 1.x | 3.3+ |

{%
    include next-prev-page-links.html
    prev=page.prev
    next=page.next
%}