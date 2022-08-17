---
title: Narrowing
layout: docs
permalink: /zh/docs/handbook/2/narrowing.html
oneline: "Understand how TypeScript uses JavaScript knowledge to reduce the amount of type syntax in your projects."
---

想想我们有个函数叫做`padLeft`。

```ts twoslash
function padLeft(padding: number | string, input: string): string {
  throw new Error("Not implemented yet!");
}
```

如果`padding`是`number`类型，它会将其视为我们要添加到“输入”的空格数。
如果`padding`是`string`类型，它会在`input`之前添加`padding`数。
让我们实现当给`padLeft`传递一个`number`来`padding`时候的逻辑。

```ts twoslash
// @errors: 2345
function padLeft(padding: number | string, input: string) {
  return " ".repeat(padding) + input;
}
```

但是，我们在`padding`上得到了一个错误。
TypeScript警告我们给repeat函数（需要`number`类型）传递`number | string`类型不会达成我们班预期的，这个是对的。
换句话说，我们没有提前具体检查`padding`是否是`number`类型，我们也没有处理是`string`字符串的情况，让我们来处理。
```ts twoslash
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

如果这看起来像是无趣的 JavaScript 代码，那就是重点。
除去我们防止我们的padding和uput的注释，这个TypeScript代码看起来和JavaScript一样。
主要思想是TypeScript的类型系统想要让写典型的JavaScript代码尽可能容易，而无需向后弯腰获取类型安全。（有类型不匹配会提示，虽然还是要麻烦写判断，只是ts更方便了）

虽然它可能看起来不多，但实际上有很多东西。
比较像TypeScript使用静态的类型分析了运行时值的类型，它将类型分析覆盖在JavaScript的运行时控制流上，这些控制流例如有`if/else`，条件三元组，循环，真实性检查等，这些都会影响类型。

通过我们的`if`检查，TypeScript看到了`typeof padding === "number"`并且理解了这段代码叫做 _类型守卫_ 。
TypeScript遵循我们的程序可能采用的执行路径来分析给定位置的值的最具体的可能类型。
它看到这些特殊的检查（叫做 _类型守卫_）和赋值，将类型细化为比声明的更具体的类型的过程成为 _缩小_ 。
在很多编辑器中，我们可以观察这些类型的变化，我们甚至在这个网站的示例中这样做。

```ts twoslash
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
    //                ^?
  }
  return padding + input;
  //     ^?
}
```

有几种不同的结构TypeScript可以理解来缩小范围。

## 1. `typeof` 类型守卫

正如我们看见的，JavaScript支持`typeof`操作符，它可以提供我们在运行时的值类型的一些非常基本的信息。
TypeScript期待这个操作符返回一些特定字符串的集合：

- `"string"`
- `"number"`
- `"bigint"`
- `"boolean"`
- `"symbol"`
- `"undefined"`
- `"object"`
- `"function"`

就像我们在`padLeft`里看见的，这个操作符在很多JavaScript库也经常出现，并且TypeScript可以在不同的分支理解他们来缩小类型。

在TypeScript里，检查`typeof`返回的值是一种类型守卫。
因为TypeScript编码了`typeof`对于不同值的操作方式，所以它知道一些JavaScript的"怪癖"。
例如，注意上面返回值列表，`typeof`不会返回字符串`null`。再看下下面的例子：

```ts twoslash
// @errors: 2531
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
}
```

在`printAll`函数里，我们尝试去检查`strs`是否是一个对象来看他是不是是一个数组类型（强调下JavaScript里数组是对象类型）。但是在JavaScript值得发现的是，`typeof null`返回值是`"object"`！
这个就是历史上一个不幸的意外。

拥有丰富经验的JavaScript用户可能不会惊讶，但是不是所有人见过JavaScript里的这种代码，幸运的是在ts里代码里上面会提示报错（就知道还要`null === null`等的额外判断l），我们的`strs`只被缩小成了`string[] | null`，而不是`string[]`（在js里可能就会忽略这种错误检查不出来）。

这个就是对我们所谓的“真实性”检查的一个很好的解释。（检查出strs可能为null的真实性）

# 2. 真实性缩小

真实性可能不是一个你能在字典里找到的值，但在JavaScript你会听到关于它的很多东西。

在JavaScript里，我们可以在条件判断用任何值，例如`&&`，`||`，`if`判断，布尔否定（`!`）等等。
例如，`if`判断条件不要求条件值一直返回`boolean`类型。


```ts twoslash
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`;
  }
  return "Nobody's here. :(";
}
```

在JavaScript里，像`if`这样的构造会首先将它们的条件判断"强制转换"转换为`boolean`类型来理解它们，然后基于它们是否为`true`或`false`来选择走向分支。
像下面的值

- `0`
- `NaN`
- `""` (the empty string)
- `0n` (the `bigint` version of zero)
- `null`
- `undefined`

所有被胁迫转为`false`，然后其他的值被胁迫转为`true`。
你总是可以通过运行`Boolean`函数将一个值胁迫转为`boolean`类型，或者通过简短的`!!`双布尔否定。（后者有好处是TypeScript会将结果判断为文本布尔类型`true`，而Boolean函数只会判断为`boolean`布尔值）。

```ts twoslash
// both of these result in 'true'
Boolean("hello"); // type: boolean, value: true
!!"world"; // type: true,    value: true
```

利用这种行为很流行，特别是对于像`null`或者`undefined`这些值的守卫很有用。
作为例子，我们在`printAll`函数上尝试使用

```ts twoslash
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  }
}
```

你将会注意到我们已经摆脱了检查`strs`是否是真实的检查。
这至少可以防止我们在运行以下代码时出现以下可怕的错误：

```txt
TypeError: null is not iterable
```

时刻注意，对原始类型进行的真实性检查通常**容易出错**。
作为一个例子，考虑在写`printAll`的时候另一种尝试

```ts twoslash {class: "do-not-do-this"}
function printAll(strs: string | string[] | null) {
  // !!!!!!!!!!!!!!!!
  //  DON'T DO THIS!
  //   KEEP READING
  // !!!!!!!!!!!!!!!!
  if (strs) {
    if (typeof strs === "object") {
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }
}
```

我们将整个函数的主体用一个真实性检查包裹起来，但是这个有一个微妙的缺点：我们不能正确处理空字符串的情况。

TypeScript这里根本不会伤害我们，但如果你对JavaScript不太熟悉，这是值得注意的行为。
TypeScript可以经常帮你提前检查bug，但是如果你选择 _忽视_ 一个值，它可以做的只有这么多，而不会过于严格规范。（就是像上面的代码情况TypeScript只能做一部分，空字符串出错了是检查不出来的要自己看）
如果您愿意，您可以确保使用 "linter" 处理此类情况。（是eslint这些额外风格层面工具？）

最后一个缩小真实性的方法是布尔否定符`!`来筛选出错误分支。

```ts twoslash
function multiplyAll(
  values: number[] | undefined,
  factor: number
): number[] | undefined {
  if (!values) {
    return values;
  } else {
    return values.map((x) => x * factor);
  }
}
```

## 2.1 相等性缩小

TypeScript也可以使用`switch`声明和像`===`,`!==`,`==`和`!=`这些相等性检查来缩小类型。
例如：

```ts twoslash
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // We can now call any 'string' method on 'x' or 'y'.
    x.toUpperCase();
    // ^?
    y.toLowerCase();
    // ^?
  } else {
    console.log(x);
    //          ^?
    console.log(y);
    //          ^?
  }
}
```

当我们在上面例子检查`x`和`y`是否相等，TypeScript知道他们的类型也要必须相等。
因为`string`是`x`和`y`的共同类型，TypeScript知道`x`和`y`在第一个分支必须都是`string`类型。

相等性检查对于文本值（针对于变量值）也同样有效。
在我们真实性缩小章节里，我们写了一个容易出错的`printAll`函数，因为他是用的`if(strs)`真实性缩小，恰巧不能正确处理空字符串的情况。
那转过头来，我们可以针对性检查排除`null`类型，滨区给TypeScript正确的从`strs`的类型里移除了`null`。

```ts twoslash
function printAll(strs: string | string[] | null) {
  if (strs !== null) {
    if (typeof strs === "object") {
      for (const s of strs) {
        //            ^?
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
      //          ^?
    }
  }
}
```

JavaScript的宽松检查`==`和`!=`也能被正确缩小。
如果你不熟悉js，那告诉你检查一个东西是否`== null`实际上不只会检查值是否未`null` — 它实际上也会潜在检查`undefined`（`null == undefined` 为true）。
对`== undefined`也是同样的：它会检查一个值是是否为`null`或`undefined`。（那看下面的`!=`同时去除了`null`和`undefined`，比较好用）

```ts twoslash
interface Container {
  value: number | null | undefined;
}

function multiplyValue(container: Container, factor: number) {
  // Remove both 'null' and 'undefined' from the type.
  if (container.value != null) {
    console.log(container.value);
    //                    ^?

    // Now we can safely multiply 'container.value'.
    container.value *= factor;
  }
}
```

## 2.2 `in`操作符缩小

JavaScript有一个操作符来判断一个对象是否有一个名字的属性值：`in`操作符。
TypeScript将这种方式算作来缩小潜在类型的一种方式。

例如，代码块`"value" in x`里，`"value"`是一个文本字符串并且`x`是一个联合类型。
然后“正确”分支缩小了x的类型，要求要么有可选要么有必须属性`value`；然后“错误”分支缩小类型，要求有个可选或者没有属性`value`。

```ts twoslash
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }

  return animal.fly();
}
```

在可选属性上用`in`判断将在缩小的两侧都出现，例如，一个人类可以同时有用和飞翔（带上特定装备）并且因此应该在`in`检查后的“正确”和“错误”分支都存在：

<!-- prettier-ignore -->
```ts twoslash
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };

function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) {
    animal;
//  ^?
  } else {
    animal;
//  ^?
  }
}
```

## 2.3 `instanceof` 缩小

JavaScript有一个操作符来检查一个值是否是另一个值的"instance"继承关系。
更具体来说，在JavaScript里`x instanceof Foo`会检查 _原型链_ 判断`Foo.prototype`是否有`x`。
这里我们不会深入研究，因为当我们讨论类的时候会加深这些概念，原型链对于哪些可以被`new`操作符构建的值很有用。
真如你猜测的，`instanceof`也是一种类型守卫，并且TypeScript缩小会发生在被`instanceof`守卫的分支。


```ts twoslash
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString());
    //          ^?
  } else {
    console.log(x.toUpperCase());
    //          ^?
  }
}
```

## 2.4 赋值

正如我们前面提到的，当我们给任意变量赋值的时候，TypeScript会检查右边的值，并且给左边变量恰当的缩小。

```ts twoslash
let x = Math.random() < 0.5 ? 10 : "hello world!";
//  ^?
x = 1;

console.log(x);
//          ^?
x = "goodbye!";

console.log(x);
//          ^?
```

注意这些赋值都是有效的。
及时我们第一次赋值我们把观察的`x`的值改为`number`类型，我们仍然可以把`string`赋予给`x`。
这是因为`x`的 _声明类型_ — `x`的类型初始是`string | number`，然后重新赋值的类型会对声明值检查判断。

如果我们给`x`赋予`boolean`的值，我们会看到报错，因为它不是声明类型的一部分。

```ts twoslash
// @errors: 2322
let x = Math.random() < 0.5 ? 10 : "hello world!";
//  ^?
x = 1;

console.log(x);
//          ^?
x = true;

console.log(x);
//          ^?
```

## 2.5 控制流分析

到此为止，我们已经看了TypeScript通过特定分支来缩小类型的基本例子。
但是还有一些情况不是简单的从`in`和`while`或者条件判断的类型守卫中对每个类型进行判断。（说的高深，就是return等，有控制流）
看下面例子

```ts twoslash
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

`padLeft`从第一个`if`块中有return语句，
然后TypeScript就能分析整个代码然后判断剩余代码部分(`return padding + input;`)当`padding`是`number`类型的时候是不能 _到达的_
因此，TypeScript能够在剩余的函数体中从`padding`中移除`number`类型（从`string | number` 缩小到 `string`）。

这种基于可到达性的代码分析叫做 _控制流分析_ ，并且TypeScript用这种流分析来缩小类型。TypeScript 在遇到类型保护和赋值时使用这种流分析来缩小类型。（所以说前面的缩小也是控制流原理？）
当一个变量被分析了。控制流可以拆分和合并一次接着一次，并且变量可以在每个不同点被观察到有不同的类型。（看下面例子）

```ts twoslash
function example() {
  let x: string | number | boolean;

  x = Math.random() < 0.5;

  console.log(x);
  //          ^?

  if (Math.random() < 0.5) {
    x = "hello";
    console.log(x);
    //          ^?
  } else {
    x = 100;
    console.log(x);
    //          ^?
  }

  return x;
  //     ^?
}
```

## 2.6 使用类型预测

我们已经从现有的JavaScript结构中处理了缩小的问题，然而有时候你想要对类型在里的代码中如何改变的有更直观的控制。

要定义一个用户定义的类型守卫，我们仅仅需要定义一个函数，让他的返回值是一个 _类型预测_：

```ts twoslash
type Fish = { swim: () => void };
type Bird = { fly: () => void };
declare function getSmallPet(): Fish | Bird;
// ---cut---
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

`pet is Fish`在这个例子里使我们的类型预测。
一个预测的形式是`parameterName is Type`，这里面`parameterName`必须是现有函数结构的一个一个参数名。

任何时候当`isFish`函数被调用传入变量，TypeScript会在变量兼容正确的时候缩小这个变量到特定的类型。

```ts twoslash
type Fish = { swim: () => void };
type Bird = { fly: () => void };
declare function getSmallPet(): Fish | Bird;
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
// ---cut---
// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet();

if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

注意TypeScript不仅仅在`if`分支直到`pet`是一个`Fish`类型；
它还知道在`else`分支里，你 _没有_ 一个`Fish`类型，所以你必须有一个`Bird`类型。

你看会用类型守卫`isFish`来从数组中筛选出`Fish | Bird`并且获得一个`Fish`的数组：
```ts twoslash
type Fish = { swim: () => void; name: string };
type Bird = { fly: () => void; name: string };
declare function getSmallPet(): Fish | Bird;
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
// ---cut---
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];
const underWater1: Fish[] = zoo.filter(isFish);
// or, equivalently
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];

// The predicate may need repeating for more complex examples
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
  if (pet.name === "sharkey") return false;
  return isFish(pet);
});
```

此外，类可以 [使用 `this is Type`](/docs/handbook/2/classes.html#this-based-type-guards) 来缩小它们的类型。

# 3. 可区分联合类型

我们为止看到的所有类型关注于缩小那些像`string`，`boolean`和`number`简单类型的单个变量。
虽然和这个很常见，但大多数时候在JavaScript里我们会处理稍微复杂的结构。

举例来说，假设我们正在尝试对园形和正方形等形状进行编码。
圆形可以关注他们的半径，正方形可以关注他们的变长。
我们将会用`kind`来告诉我们会处理哪种形状。
首先尝试定义`Shape`。

```ts twoslash
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}
```

注意我们正在使用文本类型的联合：`"circle"`和`"square"`来告诉我们是否应该将形状分开处理成圆形或正方形。
通过使用`"circle" | "square"`而不是`string`，我们可以防止错误拼写的问题。

```ts twoslash
// @errors: 2367
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}

// ---cut---
function handleShape(shape: Shape) {
  // oops!
  if (shape.kind === "rect") {
    // ...
  }
}
```

我们可以写一个`getArea`函数来
我们可以编写一个 `getArea` 函数，根据它是处理圆形还是正方形来应用正确的逻辑。
我们首先尝试处理圆形。

```ts twoslash
// @errors: 2532
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}

// ---cut---
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
}
```

<!-- TODO -->

在[`strictNullChecks`](/tsconfig#strictNullChecks)配置下会报错—是正确的，因为`radius`可能是未定义的。
但是如果我们对`kind`属性进行正确性检查呢？

```ts twoslash
// @errors: 2532
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}

// ---cut---
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
  }
}
```

嗯....，TypeScript还是不知道如何处理这个。
我们已经达到了比我们类型检查器更了解我们的值的地步。
我们可以尝试使用非空断言（一个`!`在`shape.radius`之后）（之前[Everyday Types](/zh/docs/handbook/2/everyday-types.html#non-null-assertion-operator-postfix-)讲了非空断言，意思是前面的整体不等于`undefined`，`x!.toFixed()`是说明x不是`undefined`）来告诉`shap.radius`一定存在。

```ts twoslash
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}

// ---cut---
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius! ** 2;
  }
}
```

但是这个不理想。
我们必须使用非空断言(`!`)来迫使TypeScript指导`shape.radius`是被定义了，但是如果开始在代码里迁移，这样很容易出错。
另外，当我们关闭[`strictNullChecks`](/tsconfig#strictNullChecks)选项，我们可能意外的使用`undefined`的一些字段。（因为在读取它们时TypeScript是假定可选属性始终存在的）。
我们肯定做的更好。

编码使用`Shape`类型的问题在于，类型检查器无法根据`kind`属性知道是否存在`radius`或者`sideLength`。
我们需要告诉类型检查器 _我们_ 知道的东西。
考虑到这一点，让我们再次定义`Shpae`。

```ts twoslash
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;
```

这里我们已经正确的把`Shpae`分别定义，成为有不同`kind`属性的两种类型，但是`radius`和`sideLength`在他们对应的类型定义成必选属性。

让我们看看当我们尝试获取`Shpae`的`radius`是会发生什么。

```ts twoslash
// @errors: 2339
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;

// ---cut---
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
}
```

向我们定义的第一个`Shape`类型，我们还是有报错。
当`radius`是可选的，我们得到一个报错（开启[`strictNullChecks`](/tsconfig#strictNullChecks)选项），因为TypeScript不能告诉推断这个可选`radius`是否存在。
现在`Shpae`是一个联合类型，TypeScript告诉我们`Shape`可能是`Square`，并且`Square`类型没有定义`radius`属性！
两种解释都是正确的，但是只有对于`Shape`的联合类型编码会在无论是否开启[`strictNullChecks`](/tsconfig#strictNullChecks)选项进行报错。

如果我们再次尝试检查`kind`属性？

```ts twoslash
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;

// ---cut---
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
    //               ^?
  }
}
```

摆脱报错了！
当联合类型里的每个乐行有一个公共的文本类型属性（这里的`kind`），TypeScript认为这个联合类型是 _可区分的联合类型_，并且可以从联合类型的成员里缩小出类型。

这种情况下，`kind`是一个公共属性（被认为是`Shape`的 _可区分_ 属性）。
检查`kind`属性是否是`"circle"`可以摆脱`Shpae`里的哪些`kind`不是`"circle"`的类型。
这样把`Shape`类型缩小为`Circle`类型。

这种检查对于`switch`语句同样有效。
现在让我们尝试写我们完整的`getArea`函数，避免使用任何`!`等非空断言。

```ts twoslash
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;

// ---cut---
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    //                 ^?
    case "square":
      return shape.sideLength ** 2;
    //       ^?
  }
}
```

这里重点部分是我们编码`Shape`类型的方式。
给TypeScript交流正确的信息 — 说明`Circle`和`Square`是两个真正不同的类型，通过特定的`kind`字段判断 — 是至关重要的。
这样做可以让我们写拥有类型安全的的TypeScript代码，并且看起来和我们要在JavaScript里写的方式不同的代码一样（就是TypeScript利用报错强制要你写好规范正确的代码，之后和js的规范代码一致）。
从此，类型系统能够做“正确”的事情，并且在`switch`声明的各个分支里判断出正确类型。

> 顺便说一句，尝试在上面例子里玩耍，然后移除一些`return`关键字。
> 你将会看到类型检查也可以帮助避免一些bugs，当你意外的跳转到`switch`声明的不同作用域里。

可区分的联合类型不仅仅是区分圆形和正方形，其他还是很有用的。
它们非常适合在 JavaScript 中表示任何类型的**消息传递方案**，就像你通过网络发送消息（客户端和服务端通信） ，或者在状态管理框架里编码差异更新等场景。

# 4. `nerver`类型

当缩小内省的时候，你可以减少你联合类型的选项，直到你删除了所有可能的类型并且什么都没有剩下。
在那种情况，TypeScript将会用一个`nerver`类型来代表一个不应该存在的状态。

# 5. 穷举的检查

`never`类型可以赋予给任何类型；然后，其他类型不能赋予给`never`类型的值（除了`never`本身）。这意味着你可以使用缩小类型并且依赖`never`在"switch"语句里出现来做穷举检查。

例如在我们的`getArea`函数最后加一个`default`，会尝试把shape参数变成`never`类型，这种情况当所有可能的值已经被处理最后出现。（不写_exhaustiveCheck: never，ts也会帮你推断为never，就不能赋予任何其他类型值了，本质上是shape赋予_exhaustiveCheck，导致他也变成never）

```ts twoslash
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}
// ---cut---
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```
给`Shpae`联合类型增加一个成员，会造成TypeScript报错：（场景就是加入写代码第一个版本穷举default尽了，后来加新需求，加了一个类型，ts会帮你检查你没有处理triangle的情况）

```ts twoslash
// @errors: 2322
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}
// ---cut---
interface Triangle {
  kind: "triangle";
  sideLength: number;
}

type Shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```
