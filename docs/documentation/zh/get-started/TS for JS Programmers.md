---
title: TypeScript for JavaScript Programmers
short: TypeScript for JS Programmers
layout: docs
permalink: /zh/docs/handbook/typescript-in-5-minutes.html
oneline: Learn how TypeScript extends JavaScript
---

TypeScript与JavaScript拥有不一般的关系。TypeScript提供了JavaScript的全部特性，并且提供了JavaScript更高的一层内容：TypeScript的类型系统。

例如，JavaScript提供了语言原始类型`string`和`number`，但是它不会检查你是否持续性的保持这个变量类型，但是TypeScript有这个能力，

这个意味着你现存的JavaScript也是TypeScript代码。TypeSript的主要好处是它可以在你的代码里突出报错你的意外行为，降低出bug的几率。

这个指导练习提供了TypeScript的一个简短预览，专注于类型系统。

## 推断类型

TypeScript知道JavaScript会在很多情况给你自动生成类型。
例如创建一个变量并且赋予特定的值，TypeScript也会用这个值作为它的类型。


```ts twoslash
let helloWorld = "Hello World";
//  ^?
```

通过理解JavaScript如何工作，TypeScript可以构建一个类型系统，这个系统接收JavaScript代码但是拥有类型。上面这种方式提供了一个一种类型系统，但是你不需要再你的代码里再额外写类型代码。这就是TypeScript在上面例子里如何知道`helloworld`是一个字符串`string`。

你可能在Visual Studio Code里已经写过JavaScript代码， 并且编辑器有一定自动补全功能。Visual Studio Code也可以写TypeScript进而让你更轻松的写出JavaScript（ts也会转译后变成js）

## 定义类型

你可以在JavaScript里使用多种设计模式。然而，有些设计模式让自动推断类型变得很困难（例如那些使用动态编程的设计模式）。为了处理这些类型，TypeScript支持对JavaScript语言的拓展，这样提供一些方式让你告诉TypeScript某些数据的类型是什么。

例如，为了创建一个有推断类型的对象，包含`name:string`和`id:number`两个属性，你可以像下面这样写：

```ts twoslash
const user = {
  name: "Hayes",
  id: 0,
};
```

你可以像下面显式的用`interface`定义描述一个对象的类型样式。

```ts twoslash
interface User {
  name: string;
  id: number;
}
```

然后你可以定义一个JavaScript对象来符合你的新定义的`interface`，通过在一个变量声明之后类似`: TypeName`的语法。

```ts twoslash
interface User {
  name: string;
  id: number;
}
// ---cut---
const user: User = {
  name: "Hayes",
  id: 0,
};
```

如果你提供了一个对象，但是不符合你提供的interface，TypeScript会警告你：

```ts twoslash
// @errors: 2322
interface User {
  name: string;
  id: number;
}

const user: User = {
  username: "Hayes",
  id: 0,
};
```

因为JavaScript支持class和面向对象编程，所以TypeScript也支持。你可以用interface声明来定义class：

```ts twoslash
interface User {
  name: string;
  id: number;
}

class UserAccount {
  name: string;
  id: number;

  constructor(name: string, id: number) {
    this.name = name;
    this.id = id;
  }
}

const user: User = new UserAccount("Murphy", 1);
```

你可以使用interface来给函数的入参和出参注释类型：

```ts twoslash
// @noErrors
interface User {
  name: string;
  id: number;
}
// ---cut---
function getAdminUser(): User {
  //...
}

function deleteUser(user: User) {
  // ...
}
```

在JavaScript里，已经提供了几部分少数原生类型：`boolean`,`bigint`,`null`,`number`,`string`,`symbol`和`undefined`，你可以在interface里用这些类型。Typescript把这个列表拓展了一点，例如`any`（允许任何类型）,[`unknown`](/play#example/unknown-and-never)（保证某个人用这个类型的时候需要定义这个类型是什么）, [`never`](/play#example/unknown-and-never) （这种类型不可能出出现），和`void`（一种函数会返回`undefined`或者不会返回值）。

你将会看到有两种语法构建类型：[Interfaces and Types](/play/?e=83#example/types-vs-interfaces)。你应该更会倾向于`interface`。在某些特殊场景才会用`Type`。

## 组合类型

有了TypeScript，你可以通过联合简单的类型来创建复杂的类型。有两种流行的方式来创建：用unions或者用generics。

### Unions（联合）

有了union，你可以声明一个类型表示多种类型中的一个。例如，你可以描述一个`boolean`类型，这个类型要么是`true`要么是`false`：

```ts twoslash
type MyBool = true | false;
```

_注意：_ 如果你鼠标hover在`MyBool`变量上，你将会看到他的类型是`boolean`。这个是结构类型系统的特性。更多特性如下。

一个流行的union类型使用场景是描述一个`string`或`number`的[literals](/docs/handbook/2/everyday-types.html#literal-types)的集合，表明他有哪一些允许的值:

```ts twoslash
type WindowStates = "open" | "closed" | "minimized";
type LockStates = "locked" | "unlocked";
type PositiveOddNumbersUnderTen = 1 | 3 | 5 | 7 | 9;
```

Union也提供了一种方式来处理不同的类型。例如，你可能有一个函数的参数可能是`array`或者`string`

```ts twoslash
function getLength(obj: string | string[]) {
  return obj.length;
}
```

为了了解一个变量的类型，使用`typeof`:

| Type      | Predicate                          |
| --------- | ---------------------------------- |
| string    | `typeof s === "string"`            |
| number    | `typeof n === "number"`            |
| boolean   | `typeof b === "boolean"`           |
| undefined | `typeof undefined === "undefined"` |
| function  | `typeof f === "function"`          |
| array     | `Array.isArray(a)`                 |

例如，你可以构造一个函数，先判断入参是字符串还是数组，再进行处理，返回不同的值。

<!-- prettier-ignore -->
```ts twoslash
function wrapInArray(obj: string | string[]) {
  if (typeof obj === "string") {
    return [obj];
//          ^?
  }
  return obj;
}
```

### Generics（泛型）

Generics给类型提供变量（意思是变化改变type或interface的内容）。一个普遍的例子是数组。一个数组没有泛型可能包含任何类型值。一个有泛型的数组可以描述数组包含的值类型。

```ts
type StringArray = Array<string>;
type NumberArray = Array<number>;
type ObjectWithNameArray = Array<{ name: string }>;
```

你可以自己定义一个变量并使用泛型：

```ts twoslash
// @errors: 2345
interface Backpack<Type> {
  add: (obj: Type) => void;
  get: () => Type;
}

// 这一行是个简写告诉Typescript有一个
// 常量叫做`backpack`，不要担心他是哪里来的（意思是全局常量或者declare是不用管）
declare const backpack: Backpack<string>;

// `object`变量是字符串类型，因为他在上面声明了作为`Backpack`的变量动态部分
const object = backpack.get();

// 因为backpack的变量部分是字符串类型，你不能传递数字给add函数
backpack.add(23);
```

## 结构化的类型系统

Typescript的一个核心原则是，类型检查专注于值value拥有的 _形状shape_ 。这个有时候叫做“鸭子类型”（一个东西鸭子叫或者鸭子走路就是鸭子）或者“结构化类型”

在一个结构化的类型系统，如果两个对象有相同的形状，他们会被认为是相同的type类型。

```ts twoslash
interface Point {
  x: number;
  y: number;
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}

// logs "12, 26"
const point = { x: 12, y: 26 };
logPoint(point);
```

上面代码`point`变量从来没有被声明到`Point`类型。然而，Typescript在类型检查的时候比较了`point`和`Point`的形状。他们有相同的形状，所以代码通过。（Ts自动检查形状）

类型匹配原则只需要子结构满足主结构的某些属性（下面例子）。

```ts twoslash
// @errors: 2345
interface Point {
  x: number;
  y: number;
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}
// ---cut---
const point3 = { x: 12, y: 26, z: 89 };
logPoint(point3); // logs "12, 26"

const rect = { x: 33, y: 3, width: 30, height: 80 };
logPoint(rect); // logs "33, 3"

const color = { hex: "#187ABF" };
logPoint(color);
```

classes类和object对象的形状没有本质差别（两者可以互相认形状）：

（比如下面例子把new VirtualPoint的类实例和interface Point自动识别形状了）

```ts twoslash
// @errors: 2345
interface Point {
  x: number;
  y: number;
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}
// ---cut---
class VirtualPoint {
  x: number;
  y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

const newVPoint = new VirtualPoint(13, 56);
logPoint(newVPoint); // logs "13, 56"
```

如果object或者class有所有需求的属性，TypeScript将会认为他们相互匹配，忽略一些派生额外属性等需求。

## 下一步

这个是对TypeScript日常使用的语法简单概览，从这里开始，你可以：

- 从头到尾阅读完整的[手册](/docs/handbook/intro.html) (30m)
- 探索练习场例子 [Playground examples](/play#show-examples)
