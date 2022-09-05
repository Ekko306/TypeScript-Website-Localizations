---
title: Typeof Type Operator
layout: docs
permalink: /zh/docs/handbook/2/typeof-types.html
oneline: "Using the typeof operator in type contexts."
---

## `typeof`类型操作符

JavaScript里已经有`typeof`操作符了，你可以在 _表达式_ 上下文使用：

```ts twoslash
// Prints "string"
console.log(typeof "Hello world");
```

TypeScript扩展了`typeof`操作符，你可以在 _type_ 上下文推断出一个值或属性的 _类型_：

```ts twoslash
let s = "hello";
let n: typeof s;
//  ^?
```

这个对于基本类型不是很有用，但是和其他操作符，你可以使用`typeof`方便地表达许多模式。
例如，让我们看一下预定义类型`RetrunType<T>`（内部定义直接使用的）。
它接受一个 _函数类型_，并且返回函数的返回值类型（但下面例子传入的是type的类型别名，表示函数类型）：

```ts twoslash
type Predicate = (x: unknown) => boolean;
type K = ReturnType<Predicate>;
//   ^?
```
如果我们想要在函数名上使用`ReturnType`，我们会看见指导性报错：

```ts twoslash
// @errors: 2749
function f() {
  return { x: 10, y: 3 };
}
type P = ReturnType<f>;
```

记住 _值_ 和 _类型_ 不是同一个事情，
为了推断出 _值_`f`的类型，我们使用`typeof`：

```ts twoslash
function f() {
  return { x: 10, y: 3 };
}
type P = ReturnType<typeof f>;
//   ^?
```

### 限制

TypeScript有意限制了您可以使用`typeof`的表达式类型。

具体来说，在标识符（即变量名）或其属性上使用`typeof`是唯一合法的。
这样有助于避免编写您认为正在执行但不是代码的混乱陷阱：
（就是下面msgbox(...)相当于是函数执行了，但ts里获取类型应该用ReturnType，这个叫做typeof限制了你）

```ts twoslash
// @errors: 1005
declare const msgbox: () => boolean;
// type msgbox = any;
// ---cut---
// Meant to use = ReturnType<typeof msgbox>
let shouldContinue: typeof msgbox("Are you sure you want to continue?");
```
