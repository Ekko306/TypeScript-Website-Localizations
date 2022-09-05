---
title: Indexed Access Types
layout: docs
permalink: /zh/docs/handbook/2/indexed-access-types.html
oneline: "Using Type['a'] syntax to access a subset of a type."
---

我们可以使用一个 _索引访问类型_ 来查找另一种类型的特定属性：

```ts twoslash
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"];
//   ^?
```

索引类型本身是一个类型，所以方括号里我们完全可以使用联合，`keyof`或者其他操作之后的类型：

```ts twoslash
type Person = { age: number; name: string; alive: boolean };
// ---cut---
type I1 = Person["age" | "name"];
//   ^?

type I2 = Person[keyof Person];
//   ^?

type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName];
//   ^?
```

如果您尝试索引不存的索引，您甚至会看到错误：

```ts twoslash
// @errors: 2339
type Person = { age: number; name: string; alive: boolean };
// ---cut---
type I1 = Person["alve"];
```

另一个使用任意类型索引的例子是使用`number`来获取数组元素的类型。
我们可以结合使用`typeof`来方便的获取数组文字的元素类型：

```ts twoslash
const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];

type Person = typeof MyArray[number];
//   ^?
type Age = typeof MyArray[number]["age"];
//   ^?
// Or
type Age2 = Person["age"];
//   ^?
```

你只能在索引的时候用这些类型，意味着你不能使用`const`来进行变量引用：

```ts twoslash
// @errors: 2538 2749
type Person = { age: number; name: string; alive: boolean };
// ---cut---
const key = "age";
type Age = Person[key];
```

然而，你可以使用类型别名来实现相似样式的重构：

```ts twoslash
type Person = { age: number; name: string; alive: boolean };
// ---cut---
type key = "age";
type Age = Person[key];
```
