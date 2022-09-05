---
title: Keyof Type Operator
layout: docs
permalink: /zh/docs/handbook/2/keyof-types.html
oneline: "Using the keyof operator in type contexts."
---

## `keyof`类型操作符

`keyof`操作传入一个对象类型，然后返回它键值的字符串或者数字文本联合类型。
下面的`P`类型和"x" | "y"一样：

```ts twoslash
type Point = { x: number; y: number };
type P = keyof Point;
//   ^?
```

如果类型有`string`或者`number`的下标签名，`keyof`会返回哪些下标签名：

```ts twoslash
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;
//   ^?

type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
//   ^?
```

注意在这个例子里，`M`是`string | number` -- 这是因为JavaScript对象键值总会转换成字符串，所以`obj[0]`总是和`obj["0"]`相同。

`keyof`类型在与映射类型（mapped types）结合使用时变得特别有用，我们稍后会了解。