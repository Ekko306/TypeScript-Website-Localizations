---
title: TypeScript for JavaScript Programmers
short: TypeScript for JS Programmers
layout: docs
permalink: /docs/handbook/typescript-in-5-minutes.html
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

## 联合类型

有了TypeScript，你可以通过联合简单的类型来创建复杂的类型。有两种流行的方式来创建：用unions或者用generics。

### Unions

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

To learn the type of a variable, use `typeof`:

| Type      | Predicate                          |
| --------- | ---------------------------------- |
| string    | `typeof s === "string"`            |
| number    | `typeof n === "number"`            |
| boolean   | `typeof b === "boolean"`           |
| undefined | `typeof undefined === "undefined"` |
| function  | `typeof f === "function"`          |
| array     | `Array.isArray(a)`                 |

For example, you can make a function return different values depending on whether it is passed a string or an array:

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

### Generics

Generics provide variables to types. A common example is an array. An array without generics could contain anything. An array with generics can describe the values that the array contains.

```ts
type StringArray = Array<string>;
type NumberArray = Array<number>;
type ObjectWithNameArray = Array<{ name: string }>;
```

You can declare your own types that use generics:

```ts twoslash
// @errors: 2345
interface Backpack<Type> {
  add: (obj: Type) => void;
  get: () => Type;
}

// This line is a shortcut to tell TypeScript there is a
// constant called `backpack`, and to not worry about where it came from.
declare const backpack: Backpack<string>;

// object is a string, because we declared it above as the variable part of Backpack.
const object = backpack.get();

// Since the backpack variable is a string, you can't pass a number to the add function.
backpack.add(23);
```

## Structural Type System

One of TypeScript's core principles is that type checking focuses on the _shape_ that values have. This is sometimes called "duck typing" or "structural typing".

In a structural type system, if two objects have the same shape, they are considered to be of the same type.

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

The `point` variable is never declared to be a `Point` type. However, TypeScript compares the shape of `point` to the shape of `Point` in the type-check. They have the same shape, so the code passes.

The shape-matching only requires a subset of the object's fields to match.

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

There is no difference between how classes and objects conform to shapes:

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

If the object or class has all the required properties, TypeScript will say they match, regardless of the implementation details.

## Next Steps

This was a brief overview of the syntax and tools used in everyday TypeScript. From here, you can:

- Read the full Handbook [from start to finish](/docs/handbook/intro.html) (30m)
- Explore the [Playground examples](/play#show-examples)
