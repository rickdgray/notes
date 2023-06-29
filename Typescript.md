---
title: Typescript
lastmod: 2023-06-22T14:57:37-05:00
---
# Typescript
## Configurations
### Basic, No Framework
Your `.tsconfig` file should look something like this.
```json
{
  "compileOnSave": false,
  "compilerOptions": {
    "outDir": "dist/js",
    "strict": true,
    "strictPropertyInitialization": false,
    "strictFunctionTypes": true,
    "noImplicitAny": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "sourceMap": false,
    "declaration": false,
    "downlevelIteration": true,
    "experimentalDecorators": true,
    "importHelpers": true,
    "target": "ES2022",
    "module": "none"
  },
  "include": [
    "src/ts/*"
  ]
}
```
The compiler options are sane defaults pulled from Angular. The key things to note are that
* The target is set to a modern ES version. Be sure to check [caniuse.com](https://caniuse.com/?search=es2022) for browser compatibility.
* Use the `.d.ts` file extension on interfaces, etc. to prevent generating empty `.js` files.
* The module "mode" is set to `none`. This effectively disables `import`/`export` keywords. The reason is that using module scoping makes importing and running in HTML more difficult without a framework. If modules are preferred, you can include a small `<script>` tag in your DOM that imports the module and runs a starter function. I don't prefer this.
To get around not using modules, you can use namespaces instead. Here is a simple interface for example:
```typescript
/// <reference path="User.d.ts" />

declare namespace School {
    export interface Class {
        Students: User[];
    }
}
```
