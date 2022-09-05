---
title: Modules
layout: docs
permalink: /zh/docs/handbook/2/modules.html
oneline: "How JavaScript handles communicating across file boundaries."
---

JavaScript有一个很长的历史来处理模块的代码。
TypeScript自2012年问世以来，已经实现了对许多这些格式的支持，但随着时间推移，社区和JavaScript规范已经融合到一种成为ES模组（或ES6模块）。你可能知道是`import`/`export`语法。

ES模组在2015年加入JavaScript规范，并且到2020年对于大多网络浏览器和JavaScript运行时有广泛支持。

作为重点，本手册将涵盖ES模块及其流行的前导CommonJS `module.exports=`的语法，并且你可以在 [Modules参考](/docs/handbook/modules.html) 中找到有关其他模块模式的信息 。

## JavaScript模组如何被定义

在TypeScript里，正如ECMAScript 2015的说明，任何文件包含顶级的`import`或者`export`被认为是模组。

相反，没有任何顶层import或export声明的文件被视为其内容在全局范围内可用的脚本（因此也可用于模块）。

模块在它们自己的作用于执行，而不是在全局作用于。
这意味着在模块中声明的变量、函数、类等在模块外部是不可见的，除非它们使用一种导出形式显式导出。
相反，要使用从不同模块导出的变量、函数、类、接口等，必须使用其中一种导入形式导出。

## 无模组

在我们开始之前，理解TypeScript如何看待模组是很重要的。
JavaScript规范声明任何没有`export`或顶级`await`的JavaScript文件都应该视为脚本而不是模块。

在脚本文件里，变量和类型被声明在全局领域共享的作用域，这个情况是假设你将使用[`outFile`](/tsconfig#outFile)编译器选项将多个文件连接到同一个输出文件中，或者你的HTML中使用多个`<script>`标签来加载这些文件（按照正确的作用域）。

如果你有一个当前没有任何`import`或`export`的文件，但你希望他被视为一个模块，可以添加下面这行（这样作用域就会隔离起来）：

```ts twoslash
export {};
```

这会将文件更改为不导出任何内容的模块。 无论你的模块目标如何，此语法都有效。

## TypeScript中的模块

<blockquote class='bg-reading'>
   <p>拓展阅读：<br />
   <a href='https://exploringjs.com/impatient-js/ch_modules.html#overview-syntax-of-ecmascript-modules'>Impatient JS (Modules)</a><br/>
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules'>MDN: JavaScript Modules</a><br/>
   </p>
</blockquote>

在TypeScript里当你写基于模组的代码是，有三个核心事情考虑：

- **语法**：你将使用什么语法import或者export东西？
- **模块解析**：模块名称（或路径）与磁盘上的文件有什么关系？
- **模块输出目标**：我发出的JavaScript模块应该是什么样子？

### ES 模块语法

一个文件可以通过`export default`来声明一个主要的导出：

```ts twoslash
// @filename: hello.ts
export default function helloWorld() {
  console.log("Hello, world!");
}
```

这个然后被一下方式导入：

```ts twoslash
// @filename: hello.ts
export default function helloWorld() {
  console.log("Hello, world!");
}
// @filename: index.ts
// ---cut---
import helloWorld from "./hello.js";
helloWorld();
```

除了默认导出，你还可以导出多个变量和函数，通过在`export`后删除`default`字段：

```ts twoslash
// @filename: maths.ts
export var pi = 3.14;
export let squareTwo = 1.41;
export const phi = 1.61;

export class RandomNumberGenerator {}

export function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}
```

这些变量或函数可以被另外文件通过`import`语法导入：

```ts twoslash
// @filename: maths.ts
export var pi = 3.14;
export let squareTwo = 1.41;
export const phi = 1.61;
export class RandomNumberGenerator {}
export function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}
// @filename: app.ts
// ---cut---
import { pi, phi, absolute } from "./maths.js";

console.log(pi);
const absPhi = absolute(phi);
//    ^?
```

### 额外的导入语法

一个导入可以使用`import {old as new}`的形式重命名：

```ts twoslash
// @filename: maths.ts
export var pi = 3.14;
// @filename: app.ts
// ---cut---
import { pi as π } from "./maths.js";

console.log(π);
//          ^?
```

你可以上面两个预发合并到一个`import`：（defalut和{}两种）

```ts twoslash
// @filename: maths.ts
export const pi = 3.14;
export default class RandomNumberGenerator {}

// @filename: app.ts
import RandomNumberGenerator, { pi as π } from "./maths.js";

RandomNumberGenerator;
// ^?

console.log(π);
//          ^?
```

你可以接收所有导出的对象然后全部放在一个命名空间里用`* as name`的形式：

```ts twoslash
// @filename: maths.ts
export var pi = 3.14;
export let squareTwo = 1.41;
export const phi = 1.61;

export function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}
// ---cut---
// @filename: app.ts
import * as math from "./maths.js";

console.log(math.pi);
const positivePhi = math.absolute(math.phi);
//    ^?
```

你可以导入一个文件并且 _不_ 包含任何变量到你当前的模组，使用`import "./file"`：

```ts twoslash
// @filename: maths.ts
export var pi = 3.14;
// ---cut---
// @filename: app.ts
import "./maths.js";

console.log("3.14");
```

这个例子里，`import`没有做任何事情，然而`math.ts`里的素有代码被执行了一遍，可能造成副作用影响其他变量。

#### TypeScript 特定的 ES 模块语法

类型可以像JavaScript变量一样的语法被导出和导入：

```ts twoslash
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };

export interface Dog {
  breeds: string[];
  yearOfBirth: number;
}

// @filename: app.ts
import { Cat, Dog } from "./animal.js";
type Animals = Cat | Dog;
```

TypeScript已经拓展了`import`语法，其中包含两个用于类型声明导入的概念：

###### `import type`

这是一个 _只能_ 导入**类型**的导入语句：

```ts twoslash
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
export type Dog = { breeds: string[]; yearOfBirth: number };
export const createCatName = () => "fluffy";

// @filename: valid.ts
import type { Cat, Dog } from "./animal.js";
export type Animals = Cat | Dog;

// @filename: app.ts
// @errors: 1361
import type { createCatName } from "./animal.js";
const name = createCatName();
```

###### 内联 `type` 导入

TypeScript 4.5 还允许以`type`为前缀的单个导入，以指示导入的引用是一种类型：

```ts twoslash
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
export type Dog = { breeds: string[]; yearOfBirth: number };
export const createCatName = () => "fluffy";
// ---cut---
// @filename: app.ts
import { createCatName, type Cat, type Dog } from "./animal.js";

export type Animals = Cat | Dog;
const name = createCatName();
```

这种特性允许像Babel、swc或esbuild这样的非TypeScript转移器知道可以安全地删除哪些导入。

#### 具有CommonJS行为的ES模块语法

TypeScript有一种ES模组已发可以 _直接_ 与CommonJS和AMD的`require`关联。_大多数情况下_ 使用ES模块的导入与那些环境中的`require`相同，但这种预发可以确保你自己写的TypeScript文件与CommonJS输出有1对1的匹配：

```ts twoslash
/// <reference types="node" />
// @module: commonjs
// ---cut---
import fs = require("fs");
const code = fs.readFileSync("hello.ts", "utf8");
```

你可以在[模组参考页](/docs/handbook/modules.html#export--and-import--require)学习这种语法的更多内容。

## CommonJS 语法

CommonJS是npm上大多模块的上传格式。即使你在上面用ES模块预发进行编写，对CommonJS语法的工作原理有一个简要的了解可以帮助你更轻松地调试。

#### Exporting

这种模块的标识符是通过名为`module`的全局变量上设置的`exports`属性导出的。

```ts twoslash
/// <reference types="node" />
// ---cut---
function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}

module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
};
```

然后这些文件可以通过`require`语句导入：

```ts twoslash
// @module: commonjs
// @filename: maths.ts
/// <reference types="node" />
function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}

module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
};
// @filename: index.ts
// ---cut---
const maths = require("maths");
maths.pi;
//    ^?
```

或者你可以使用JavaScript的解构特性简化一点：

```ts twoslash
// @module: commonjs
// @filename: maths.ts
/// <reference types="node" />
function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}

module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
};
// @filename: index.ts
// ---cut---
const { squareTwo } = require("maths");
squareTwo;
// ^?
```

### CommonJS和ES Modules模块互相操作

关于默认导入和模块命名空间对象导入之间的区别，CommonJSheES模块之间的特性可能不匹配。TypeScript有一个编译器标识，可以通过[`esModuleInterop`](/tsconfig#esModuleInterop)减少两组不同约束之间的摩擦。

## TypeScript的模块解析选项

模块解析是从`import`或者`require`语句拿到一个字符串，然后决定这个字符串指向什么文件。

TypeScript包括两种解析策略：经典的和Node的。经典的是当编译器选项[`module`](/tsconfig#module)不是`commonjs`时的默认值，包含在内是为了向后兼容。
Node策略复制了Node.js在CommonJS模式下的工作方式，并额外检查了`.ts`和`.d.ts`。

有很多TS配置标志可以影响TypeScript中的模块策略：[`moduleResolution`](/tsconfig#moduleResolution)、[`baseUrl`](/tsconfig#baseUrl)、[`paths`](/tsconfig#paths)、[ `rootDirs`](/tsconfig#rootDirs)。

有关这些策略如何运作的完整详细信息，你可以查阅[Module Resolution](/docs/handbook/module-resolution.html)。

## TypeScript的模块输出选项

影响JavaScript输出结果的有两个属性：

- [`target`](/tsconfig#target) 确定哪些 JS 功能被降级（转换为在较旧的 JavaScript 运行时中运行），哪些保持不变
- [`module`](/tsconfig#module) 确定模块之间使用什么代码进行交互

你使用哪种 [`target`](/tsconfig#target)取决于你希望在在其中运行的TypeScript代码的JavaScript运行时的可用的功能。你可能要：支持最旧的Web浏览器，最低版本的Node.js，这些环境是你希望在上面运行或者可能来自运行时的独特约束—例如Electron。

模块之间的沟通通过模组加载器实现，编译器选项[`module`](/tsconfig#module)决定了使用那个加载器。
在运行时模块加载器负责在执行模块之前定位和执行模块的依赖项。

例如，这里有个TypeScript文件使用ES模块语法，展示 [`module`](/tsconfig#module)的几个不同选项：

```ts twoslash
// @filename: constants.ts
export const valueOfPi = 3.142;
// @filename: index.ts
// ---cut---
import { valueOfPi } from "./constants.js";

export const twoPi = valueOfPi * 2;
```

#### `ES2020`

```ts twoslash
// @showEmit
// @module: es2020
// @noErrors
import { valueOfPi } from "./constants.js";

export const twoPi = valueOfPi * 2;
```

#### `CommonJS`

```ts twoslash
// @showEmit
// @module: commonjs
// @noErrors
import { valueOfPi } from "./constants.js";

export const twoPi = valueOfPi * 2;
```

#### `UMD`

```ts twoslash
// @showEmit
// @module: umd
// @noErrors
import { valueOfPi } from "./constants.js";

export const twoPi = valueOfPi * 2;
```

>请注意，ES2020 实际上与原始的 `index.ts` 相同。


您可以在 [TSConfig Reference for `module`](/tsconfig#module) 中查看所有可用选项以及它们编译出的的 JavaScript 代码的样子。

## TypeScript命名空间

TypeScript 有自己的模块格式，称为`namespace`，它早于 ES 模块标准。 这种语法对于创建复杂的定义文件有很多有用的特性，并且仍然在[indefiniteTyped](/dt) 中得到积极使用。 虽然没有被弃用，但`namespace`中的大多数功能都存在于 ES 模块中，我们建议您使用它来与 JavaScript 的方向保持一致。 您可以在 [命名空间参考页面](/docs/handbook/namespaces.html) 中了解有关命名空间的更多信息。
