# Installation

To install Zod v3:

```sh
npm install zod
```

⚠️ IMPORTANT: You must enable `strict` mode in your `tsconfig.json`. This is a best practice for all TypeScript projects.

```ts
// tsconfig.json
{
  // ...
  "compilerOptions": {
    // ...
    "strict": true
  }
}
```

#### TypeScript requirements

- Zod 3.x requires TypeScript 4.1+
- Zod 2.x requires TypeScript 3.7+
- Zod 1.x requires TypeScript 3.3+