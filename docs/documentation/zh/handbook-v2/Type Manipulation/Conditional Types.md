---
title: Conditional Types
layout: docs
permalink: /zh/docs/handbook/2/conditional-types.html
oneline: "Create types which act like if statements in the type system."
---

在大多数实用项目的核心，我们必须根据输入作出决策。
JavaScript程序也不例外，但是考虑值很容易自省（不太懂意思，翻译成中文也不太懂），这些决策也可以基于输入的类型。
_条件类型_ 帮助描述输入和输出类型之间的关系。

```ts twoslash
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}

type Example1 = Dog extends Animal ? number : string;
//   ^?

type Example2 = RegExp extends Animal ? number : string;
//   ^?
```

条件类型的形式看起来有点像JavaScript中的条件表达式 (`condition ? trueExpression : falseExpression`) ：

```ts twoslash
type SomeType = any;
type OtherType = any;
type TrueType = any;
type FalseType = any;
type Stuff =
  // ---cut---
  SomeType extends OtherType ? TrueType : FalseType;
```

当`extends`左侧的类型可以分配给右侧的类型（是否可以extends），你就可以从第一个分支（"true"分支）获取相应类型，否则从第二个分支("false"分支）获取相应类型。

从上面的例子，条件类型可能不会理解看上去很有用 — 但是我们可以从`Dog extends Animal`的正确与否，获取`number`或者`string`！
但是条件类型的强大之处在将它们与泛型一起使用。

例如，我们可以实现下面的`createLabel`函数：

```ts twoslash
interface IdLabel {
  id: number /* some fields */;
}
interface NameLabel {
  name: string /* other fields */;
}

function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw "unimplemented";
}
```

createLabel这些重载描述了一个JavaScript函数，该函数根据输入的类型进行选择。注意几点问题：

1. 如果一个库必须在其API中一遍又一遍地做出相同的选择，这将变得很麻烦。
2. 我们必须创建三个重载，一个用于 _确定_ 类型的每种情况（一个用于`string`，一个用于`number`）。对于`createLabel`可以处理的每种新类型，重载的数量会呈指数增长。

相反，我们可以使用条件类型编码这个逻辑：

```ts twoslash
interface IdLabel {
  id: number /* some fields */;
}
interface NameLabel {
  name: string /* other fields */;
}
// ---cut---
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
```

我们可以使用条件类型来简化我们的重载，变成没有重载的单个函数。

```ts twoslash
interface IdLabel {
  id: number /* some fields */;
}
interface NameLabel {
  name: string /* other fields */;
}
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
// ---cut---
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  throw "unimplemented";
}

let a = createLabel("typescript");
//  ^?

let b = createLabel(2.8);
//  ^?

let c = createLabel(Math.random() ? "hello" : 42);
//  ^?
```

### 条件类型约束

通常，条件检查会为我们提供一些新的信息。
就像类型守卫的缩小可以给我们更具体的类型，条件类型的正确分支将会进一步限制我们检查类型的泛型。

例如，我们看下面例子：

```ts twoslash
// @errors: 2536
type MessageOf<T> = T["message"];
```

在这个例子里，TypeScript错误是因为`T`不能知道拥有叫做`message`的属性。
我们可以像下面约束`T`，然后TypeScript将不会报错：

```ts twoslash
type MessageOf<T extends { message: unknown }> = T["message"];

interface Email {
  message: string;
}

type EmailMessageContents = MessageOf<Email>;
//   ^?
```

然而，当我们想要`MessageOf`接收任何值，并且当没有`message`属性的时候默认是`never`值？
我们可以通过将约束搬移出来并且使用一个条件类型：

```ts twoslash
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

interface Email {
  message: string;
}

interface Dog {
  bark(): void;
}

type EmailMessageContents = MessageOf<Email>;
//   ^?

type DogMessageContents = MessageOf<Dog>;
//   ^?
```

在正确分支里，TypeScript指导`T` _将会_ 有`message`属性。

作为另一个例子，我们也可以写一个叫做`Flatten`的类型将数组类型展平为其元素的类型，非数组不会理会原样返回：

```ts twoslash
type Flatten<T> = T extends any[] ? T[number] : T;

// Extracts out the element type.
type Str = Flatten<string[]>;
//   ^?

// Leaves the type alone.
type Num = Flatten<number>;
//   ^?
```

当`Flatten`传入数组类型，它会使用`number`的索引获取到`string[]`的元素类型。
其他情况，它会返回给的值。

### 在条件类型中推断

我们刚刚发现自己使用条件类型来应用约束，然后提取类型。
这最终成为一种常见的操作，条件类型会使它更容易。

条件类型给我们提供了一种方式使用`infer`关键字从我们在真实分支中比较的类型进行推断的方法。
例如我们可以`Flatten`中推断元素类型，而不是使用索引访问类型“手动”获取它（对比上面`type Flatten<T> = T extends any[] ? T[number] : T;`是用`T[number]`取到的）：

```ts twoslash
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
```

这里，我们使用了`infer`关键字声明性地引入了一个名为`Item`的新泛型类型变量，而不是指定如何在`true`分支检索出`T`的元素类型。
这个使我们不必考虑如何挖掘和探索我们感兴趣的类型的结构。

我们可以通过`infer`关键字写一些有用的辅助类型别名。
例如，对于简单情况，我们可以从函数类型中提取返回类型：

```ts twoslash
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
  ? Return
  : never;

type Num = GetReturnType<() => number>;
//   ^?

type Str = GetReturnType<(x: string) => string>;
//   ^?

type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>;
//   ^?
```

当我们从具有多个调用签名的类型（例如重载函数类型）进行推断时，会根据 _最后一个_ 见面判断（这看您是最宽松的包罗万象的情况）。基于很多种类型参数进行多个重载函数的推断是不可能的（只能选最后一个）。

```ts twoslash
declare function stringOrNum(x: string): number;
declare function stringOrNum(x: number): string;
declare function stringOrNum(x: string | number): string | number;

type T1 = ReturnType<typeof stringOrNum>;
//   ^?
```

## 分布式条件类型

当在泛型类型上使用条件类型的时候，当传入联合类型时他们就变成 _分布式的_。 
例如，看下面例子：

```ts twoslash
type ToArray<Type> = Type extends any ? Type[] : never;
```

如果我们在`ToArray`里传入一个联合类型，然后条件类型就会被应用到联合类型的每个成员

```ts twoslash
type ToArray<Type> = Type extends any ? Type[] : never;

type StrArrOrNumArr = ToArray<string | number>;
//   ^?
```

这里发生的事情是`StrArrOrNumArr`在下面两个属性上发生分布式：

```ts twoslash
type StrArrOrNumArr =
  // ---cut---
  string | number;
```

然后映射到联合类型的每个成员，条件判断那个是有效的：

```ts twoslash
type ToArray<Type> = Type extends any ? Type[] : never;
type StrArrOrNumArr =
  // ---cut---
  ToArray<string> | ToArray<number>;
```

结果转换成：

```ts twoslash
type StrArrOrNumArr =
  // ---cut---
  string[] | number[];
```

通常，分配性是期望的行为。
为避免这种行为，您可以用方括号将`extends`关键字的每一侧括起来。

```ts twoslash
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;

// 'StrArrOrNumArr' is no longer a union.
type StrArrOrNumArr = ToArrayNonDist<string | number>;
//   ^?
```
