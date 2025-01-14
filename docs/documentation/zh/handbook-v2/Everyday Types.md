---
title: Everyday Types
layout: docs
permalink: /zh/docs/handbook/2/everyday-types.html
oneline: "The language primitives."
---

在这一章，我们将会讨论一些你在JavaScript你使用的常见变量类型，并且解释这些类型在TypeScript里的对应使用。
这不是一个详尽的列表，并且未来章节将会描述和列举更多其他类型。

类型也可以在很多 _地方_ 出现，不只是在类型注释。
在我们学习类型本身的同时，我们还将了解可以引用这些类型以形成新结构的地方。

我们通过当你在写ts或者js代码可能遇见的最基本常见的类型开始。
这些基本类型之后会用来构建复杂类型。

## 原始类型：`字符串`,`数字`和`布尔值`

Javascript有三个很常见的[原始类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)：`字符串`，`数字`和`布尔值`。每种类型在TypeScript里有对应类型。
正如你猜想的，如果您对这些类型使用JavaScript的typeofOperator, you will看到这些名称：

- `string`代表字符串的值，例如`"Hello, world"`
- `number`对于像`42`的数字。JavaScript对于整数特定运行时的值，所以也同样没有，所以不存在`int`和`float`的比较 —— 所有东西都是`number`类型
- `boolean`对于`true`和`false`两个值

 > 类型名称`String`,`Number`,和`Boolean`（大写字母开头）是合法的，指向内置的类型，但是你代码很少会用到。 _总是_ 使用`string`,`number`或者`boolean`来定义类型。

## Arrays数组

为了表明类似`[1, 2, 3]`的数组类型，你可以使用`number[]`语法；这个预发对于任何类型都适用。（例如`string[]`表示字符串数组等等）。
你可能有些写法`Array<number>`，这两个是一样的。
当我们学到 _泛型_ 的时候会讲到更多`T<U>`的语法。

> 注意`[number]`是另外的类型；将在[元组](/docs/handbook/2/objects.html#tuple-types)章节学习。

## `any`任意类型

TypeScript也有一个特殊的类型，`any`，当你不想给一个特定的值进行类型检查和错误提示的时候使用。

当一个值的类型是`any`，你可以获取它的任意属性（取的值也是`any`类型），对它可以像函数一样调用，赋予一个任意类型的值，或者一些符合语法规则的操作：

```ts twoslash
let obj: any = { x: 0 };
// None of the following lines of code will throw compiler errors.
// Using `any` disables all further type checking, and it is assumed 
// you know the environment better than TypeScript.
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```

当你不想写一长串类型注释来说服TypeScript一行代码类型是类型正确的时候，`any`类型会非常有用。

### `noImplicitAny`

当你不指明一个类型的时候，并且TypeScript不能从上下文推断的时候，编译器会把类型默认设置为`any`。

你通常想要避免这个（要更规范，禁止用any），因为`any`类型不会被类型检查。
可以给`tsconfig.js`设置[`noImplicitAny`](/tsconfig#noImplicitAny)配置，让隐式的`any`当做ts的一个报错。

## 变量的类型注释

当你用`const`,`var`或者`let`声明变量的时候，你可以可选的增加一个类型注释来精确说明变量的类型：

```ts twoslash
let myName: string = "Alice";
//        ^^^^^^^^ Type annotation
```

> TypeScript不会使用类似`int x = 0;`的“类型在左”的声明语法。类型注释总是在类型声明 _之后_。

然而在大多数情况下，这个是不需要的
尽可能的情况下，TypeScript会尝试自动从你的代码里 _推断_ 类型。例如，一个变量的类型可以从它的初始值推断。

```ts twoslash
// No type annotation needed -- 'myName' inferred as type 'string'
let myName = "Alice";
```

对于大多数情况下，你不需要精确了解推断的规则。
如果你是新手，尝试尽可能少的在ts里写你想的类型注释—您可能会惊讶于 TypeScript 完全理解正在发生的事情时，所需的具体说明很少。

## 函数

在JavaScript里函数的原始意义是各处传递数据。
TypeScript允许你指明函数的输入和输出的类型。

### 参数类型注释

当你声明一个函数的时候，你可以在每个参数后面添加注释，声明函数需要什么类型的参数。
参数类型注释写在参数名后面：

```ts twoslash
// Parameter type annotation
function greet(name: string) {
  //                 ^^^^^^^^
  console.log("Hello, " + name.toUpperCase() + "!!");
}
```

当一个参数有类型注释，这个函数的参数会被检查：

```ts twoslash
// @errors: 2345
declare function greet(name: string): void;
// ---cut---
// Would be a runtime error if executed!
greet(42);
```

> 即使你没有给你的参数写类型注释，TypeScript也会检查是否传递了正确个数的参数。


### 返回值类型注释

你也可以给家返回值添加类型注释。
返回值类型声明在参数列表之后出现：

```ts twoslash
function getFavoriteNumber(): number {
  //                        ^^^^^^^^
  return 26;
}
```

比较像变量类型注释，你通常不需要写返回值类型注释，因为TypeScript会从函数的返回值推断，基于`return`的声明值。
在上面例子的类型注释不会改变什么东西。
有些代码显式的指明返回类型是为了写清楚文档，避免偶发的改变，或者只是个人喜好。

### 匿名函数

匿名函数和普通函数声明有点不同。
当一个函数出现在 TypeScript 可以确定如何调用它的地方时，该函数的参数会自动被赋予类型。

下面是例子：

```ts twoslash
// @errors: 2551
// No type annotations here, but TypeScript can spot the bug
const names = ["Alice", "Bob", "Eve"];

// Contextual typing for function
names.forEach(function (s) {
  console.log(s.toUppercase());
});

// Contextual typing also applies to arrow functions
names.forEach((s) => {
  console.log(s.toUppercase());
});
```

尽管参数`s`没有类型注释，TypeScript使用了`forEach`函数的类型推断，连同数组的推断类型一起，来确定了`s`类型会有的类型。

这个过程叫做 _上下文类型_ 因为函数发生的 _上下文_ 告知了它应该有的类型。

类似类型推断规则，你不需要精确学习这个如何发生的，但是理解这个让你知道有些类型注释不必要。
之后，我们将看到更多关于值出现的上下文如何影响其类型的示例。

## 对象类型

除了原生类型，你将遇见的更常见的类型是 _对象类型_。这个指任何JavaScript里有属性的值，意味着几乎js里的所有值！
为了定义对象类型，我们只用列出他们的属性和类型。

例如，这个是一个函数，接收一个类似坐标的对象：

```ts twoslash
// The parameter's type annotation is an object type
function printCoord(pt: { x: number; y: number }) {
  //                      ^^^^^^^^^^^^^^^^^^^^^^^^
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });
```

这里我们我们给参数声明，用了两个属性—`x`和`y`—两个都是`number`数字类型。
你可以使用`,`或者`;`来分隔不同的属性，然后最后一个分隔符是可选的。

每个属性值也是可以可选的。
如果你不想指明一个类型，它可以被声明为`any`类型。

### 可选属性

对象类型也可以指明属性的一些或者所有属性是可选的。
为了实现这个，在属性名后面增加一个`?`：

```ts twoslash
function printName(obj: { first: string; last?: string }) {
  // ...
}
// Both OK
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });
```
在JavaScript里，如果你尝试获取一个不存在的属性，你将会得到`undefined`而不是运行时错误。
因为这个，当你从一个可选值 _获取_ 值的时候，你要在使用前检查他是不是`undefined`的值。

```ts twoslash
// @errors: 2532
function printName(obj: { first: string; last?: string }) {
  // Error - might crash if 'obj.last' wasn't provided!
  console.log(obj.last.toUpperCase());
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }

  // A safe alternative using modern JavaScript syntax:
  console.log(obj.last?.toUpperCase());
}
```

## 联合类型

TypeScript的类型系统允许你从**现存的类型构建新的类型**，可以使用一大堆提供的方法。
现在我们知道了如何写一点类型，是时候开始用有趣的方式组合他们了。

### 定义一个联合类型

你将接触的最初组合类型方式可能是 _union_ 类型。
一个联合类型是一个从两个或更多类型组合的类型，代表这个值可能是那些值的 _任意一个_。
我们将这些类型中的每一种称为联合类型的 _成员_。 

让我们写一个函数可以操作字符串或者数字：

```ts twoslash
// @errors: 2345
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// Error
printId({ myID: 22342 });
```

### 使用联合类型工作

_提供_ 一个值匹配联合类型很容易—仅仅需要提供任何一个联合类型成员匹配的值。
如果你 _有_ 一个联合类型的值，你将如何使用它？

TypeScript只会允许union里的所有成员都满足的操作发生。
例如，如果你有一个联合类型`string | number`，你不能使用仅仅在`string`上有的方法：

```ts twoslash
// @errors: 2339
function printId(id: number | string) {
  console.log(id.toUpperCase());
}
```

解决方法是用代码 _缩小_ 联合类型的值，就像你在JavaScript没有用类型注释的写法一样。
当 TypeScript 可以根据代码的结构为某个值推断出更具体的类型时，就会发生 _缩小_。

例如，TypeScript知道只有`string`值将会有一个`typeof`的值`"string"`：

```ts twoslash
function printId(id: number | string) {
  if (typeof id === "string") {
    // In this branch, id is of type 'string'
    console.log(id.toUpperCase());
  } else {
    // Here, id is of type 'number'
    console.log(id);
  }
}
```

另外一个例子是使用类似`Array.isArray`的函数：

```ts twoslash
function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // Here: 'x' is 'string[]'
    console.log("Hello, " + x.join(" and "));
  } else {
    // Here: 'x' is 'string'
    console.log("Welcome lone traveler " + x);
  }
}
```

注意在else分支，我们不需要做任何声明事情—如果`x`不是一个`string[]`类型，则他一定是`string`类型。

有时候你将有些联合类型，然而他们所有成员有一些类似之处。
例如，数组和字符串类型都有`slice`方法。
如果每个联合类型的成员都有共同的类型，你可以不需要缩小就是用属性值：

```ts twoslash
// Return type is inferred as number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3);
}
```

> 一个 _联合类型_ 拥有这些属性的交集可能让你迷惑。
> 这个不是偶然 — 名字 _联合_ 就是从这种类型理论来的。
> _联合类型_ `number | string`是由每种类型的值并集组成。
> 注意给定两个类型，每个类型都有对应的事实，只有这些事实的 _交集_ 部分适合于集合本身的 _并集_。
> 例如，如果我们有一屋子高个子人带着帽子，然后另一个房间有西班牙的演讲者们带着帽子，在组合两个房间后，我们唯一知道的事情是他们 _每一个_ 人必须带着帽子。

## 类型别名

我们已经通过直接在类型注释列写，使用了对象类型和联合类型。（上面都是原生手写）
这是个很方便，但是当我们想要使用同一个类型多次使用并且用一个单独名字获取也很常见。

一个 _类型别名_ 就是为了和这个用途—一个给任意 _类型_ 的 _名字_。
类型别名的语法是：

```ts twoslash
type Point = {
  x: number;
  y: number;
};

// Exactly the same as the earlier example
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

你可以事实上用用类型别名来给任意类型一个名字，不只是对象类型。
例如，一个类型别名可以命名一个联合类型：

```ts twoslash
type ID = number | string;
```

注意别名 _只是_ 别名 — 你不能使用类型别名来创建同一个类型的不同/独特的“版本”。
当您使用别名时，就好像您已经编写了别名类型。
换句话说，下面这段代码可能 _看起来_ 是非法的，但根据 TypeScript 是可以的，因为这两种类型都是同一类型的别名（UserInputSanitizedString虽然取了名字，但还是可以和string比较是同一个）：

```ts twoslash
declare function getInput(): string;
declare function sanitize(str: string): string;
// ---cut---
type UserInputSanitizedString = string;

function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}

// Create a sanitized input
let userInput = sanitizeInput(getInput());

// Can still be re-assigned with a string though
userInput = "new input";
```

## 接口

一个 _接口声明_ 是另一个方式命名一个对象类型：

```ts twoslash
interface Point {
  x: number;
  y: number;
}

function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

就像我们在上面使用类型别名一样，上面例子在我们使用一个匿名对象类型时也适用。
TypeScript知识考虑我们传递给`printCoord`的 _数据结构_ — 他只会关系他有期望的属性。

### TypeAliase类型别名和Interfaces接口的区别

类型别名和接口非常相似，然后再很多情况可以两者间随意选择。
几乎`interface`的所有特性都能用`type`实现，关键区分是类型不能被再次打开然后赋予新值，而接口是总是可扩展的。

<table class='full-width-table'>
  <tbody>
    <tr>
      <th><code>Interface</code></th>
      <th><code>Type</code></th>
    </tr>
    <tr>
      <td>
        <p>拓展一个接口</p>
        <code><pre>
interface Animal {
  name: string
}<br/>
interface Bear extends Animal {
  honey: boolean
}<br/>
const bear = getBear() 
bear.name
bear.honey
        </pre></code>
      </td>
      <td>
        <p>通过\&交集符号拓展类型别名</p>
        <code><pre>
type Animal = {
  name: string
}<br/>
type Bear = Animal & { 
  honey: boolean 
}<br/>
const bear = getBear();
bear.name;
bear.honey;
        </pre></code>
      </td>
    </tr>
    <tr>
      <td>
        <p>给现有interface添加新字段</p>
        <code><pre>
interface Window {
  title: string
}<br/>
interface Window {
  ts: TypeScriptAPI
}<br/>
const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
        </pre></code>
      </td>
      <td>
        <p>一个类型别名在创建后不能被修改</p>
        <code><pre>
type Window = {
  title: string
}<br/>
type Window = {
  ts: TypeScriptAPI
}<br/>
<span style="color: #A31515"> // Error: Duplicate identifier 'Window'.</span><br/>
        </pre></code>
      </td>
    </tr>
    </tbody>
</table>

你将会在后面章节学到更多概念，所以不要担心如果你不能理解这些所有概念。

- 对于TypeScript低于4.2的版本，类型别名 [_可能_ 会有错误信息](/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWZWhfYAjABMAMwALA+gbsVjoADqgjKESytQPxCHghAByXigYgBfr8LAsYj8aQMUASbDQcRSExCeCwFiIQh+AKfAYyBiQFgOPyIaikSGLQo0Zj-aazaY+dSaXjLDgAGXgAC9CKhDqAALxJaw2Ib2RzOISuDycLw+ImBYKQflCkWRRD2LXCw6JCxS1JCdJZHJ5RAFIbFJU8ADKC3WzEcnVZaGYE1ABpFnFOmsFhsil2uoHuzwArO9SmAAEIsSFrZB-GgAjjA5gtVN8VCEc1o1C4Q4AGlR2AwO1EsBQoAAbvB-gJ4HhPgB5aDwem-Ph1TCV3AEEirTp4ELtRbTPD4vwKjOfAuioSQHuDXBcnmgACC+eCONFEs73YAPGGZVT5cRyyhiHh7AAON7lsG3vBggB8XGV3l8-nVISOgghxoLq9i7io-AHsayRWGaFrlFauq2rg9qaIGQHwCBqChtKdgRo8TxRjeyB3o+7xAA)，（就是有时候报错，有时候不报错），有时候可能在等效的匿名类型出报错信息（可能需要也可能不需要）。接口始终在类型系统中出现。（看例子里，就是低版本对于Omit等操作后的不能定位到type名称，但是高版本可以）
- 类型别名可能[不会参与声明合并，但是接口可以](/play?#code/PTAEEEDtQS0gXApgJwGYEMDGjSfdAIx2UQFoB7AB0UkQBMAoEUfO0Wgd1ADd0AbAK6IAzizp16ALgYM4SNFhwBZdAFtV-UAG8GoPaADmNAcMmhh8ZHAMMAvjLkoM2UCvWad+0ARL0A-GYWVpA29gyY5JAWLJAwGnxmbvGgALzauvpGkCZmAEQAjABMAMwALLkANBl6zABi6DB8okR4Jjg+iPSgABboovDk3jjo5pbW1d6+dGb5djLwAJ7UoABKiJTwjThpnpnGpqPBoTLMAJrkArj4kOTwYmycPOhW6AR8IrDQ8N04wmo4HHQCwYi2Waw2W1S6S8HX8gTGITsQA)。（例如上面Window声明合并）
- 接口可能只会被用作[声明对象的类型，但是不能重命名原生类型](/play?#code/PTAEAkFMCdIcgM6gC4HcD2pIA8CGBbABwBtIl0AzUAKBFAFcEBLAOwHMUBPQs0XFgCahWyGBVwBjMrTDJMAshOhMARpD4tQ6FQCtIE5DWoixk9QEEWAeV37kARlABvaqDegAbrmL1IALlAEZGV2agBfampkbgtrWwMAJlAAXmdXdy8ff0Dg1jZwyLoAVWZ2Lh5QVHUJflAlSFxROsY5fFAWAmk6CnRoLGwmILzQQmV8JmQmDzI-SOiKgGV+CaYAL0gBBdyy1KCQ-Pn1AFFplgA5enw1PtSWS+vCsAAVAAtB4QQWOEMKBuYVUiVCYvYQsUTQcRSBDGMGmKSgAAa-VEgiQe2GLgKQA)。
- 接口名字将[总会在错误信息里显示原始的形式](/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWY2Q-YAjABMAMwALA+gbsVjNXW8yxySoAADaAA0CCaZbPh1XYqXgOIY0ZgmcK0AA0nyaLFhhGY8F4AHJmEJILCWsgZId4NNfIgGFdcIcUTVfgBlZTOWC8T7kAJ42G4eT+GS42QyRaYbCgXAEEguTzeXyCjDBSAAQSE8Ai0Xsl0K9kcziExDeiQs1lAqSE6SyOTy0AKQ2KHk4p1V6s1OuuoHuzwArMagA)，但是 _只有_ 当他们被名字声明使用的时候出现。（不能用匿名，否则肯定没名字）

对于大多数情况，你可以从个人喜好选择，并且TypeScript将会告诉你是否要用另一种声明方式当你要用些额外特性的时候。如果你想要启发式，请使用`interface`直到你必须使用`type`的特性。

## 类型断言

有时你会得到关于 TypeScript 无法知道的值类型的信息。（TypeScript只会从代码层面考虑很多，但是有些情况是你自己排除了必定会出现的）

例如，如果你使用`document.getElementById`，TypeScript只会知道哪它会返回`HTMLElement`的 _某些_ 类型，但是你看知道你的对于给的ID页面总会有一个`HTMLCanvasElement`。

对于这种情况，你可以用一个 _类型断言_ 来指定更加具体的类型：

```ts twoslash
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

就像类型注释一样，类型断言也会被编译器移除并且不会影响你代码的运行时。

你也可以使用尖括号语法（不能在`.tsx`文件里），和as语法是等效的：

```ts twoslash
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```
> 提醒：因为类型断言在编译时被删除，所以没有与类型断言关联的运行时检查。
> 如果类型断言错了，不会有报错或者null生成。（就是自己定义的确定性断言错了，之后运行时的不一致报错ts是出来的）

TypeScript只会允许类型断言对于同一个类型别名的 _更精确_ 或者 _更不精确_ 进行转换。
此规则可防止“不可能”的强制，例如：

```ts twoslash
// @errors: 2352
const x = "hello" as number;
```

有时候这个规则可能会过于保守，并且不允许可能有效的更复杂的强制转换。
如果和这个发生，你可以用两次断言，首先转换为`any`（或者`unknown`， 之后会介绍），然后转换为想要的类型：

```ts twoslash
declare const expr: any;
type T = { a: 1; b: 2; c: 3 };
// ---cut---
const a = (expr as any) as T;
```

## 文字类型

对于通用的类型`string`和`number`外，我们还可以在类型位置指明 _特定的_ 字符串和数字。

这样思考的方式是思考JavaScript有两种方式声明变量。`var`和`let`都允许修改定义的变量，但是`const`不行。这个也影响了TypeScript创建文字类型的方式。（下面constantString被定义死了具体的字符串内容）

```ts twoslash
let changingString = "Hello World";
changingString = "Olá Mundo";
// Because `changingString` can represent any possible string, that
// is how TypeScript describes it in the type system
changingString;
// ^?

const constantString = "Hello World";
// Because `constantString` can only represent 1 possible string, it
// has a literal type representation
constantString;
// ^?
```

就其本身而言，文字类型并不是很有价值：

```ts twoslash
// @errors: 2322
let x: "hello" = "hello";
// OK
x = "hello";
// ...
x = "howdy";
```

变量只能有一个值并没有多大用处！

但是通过 _组合_ 文本变量到联合类型，你可以表达一个更有用的概念—例如，函数只允许特定允许值的集合：

```ts twoslash
// @errors: 2345
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
```

数字文本类型也可以同样方式工作：

```ts twoslash
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

当然你可以将他们和非文本类型组合：

```ts twoslash
// @errors: 2345
interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}
configure({ width: 100 });
configure("auto");
configure("automatic");
```

还有一种文本类型：布尔文本
只有两种布尔文本类型，正如你能猜到的的，他们就是类型`true`和`false`。
`boolean`类型本身就是`true | false`的联合类型。

### 文本接口

当你对一个变量初始化值的时候，TypeScript猜测那个对象的属性值可能之后会改变值。
例如，你可能写下面的代码：


```ts twoslash
declare const someCondition: boolean;
// ---cut---
const obj = { counter: 0 };
if (someCondition) {
  obj.counter = 1;
}
```

TypeScript不会认为给一个之前是`0`的字段赋予`1`是一个错误。
另外一个方式表述这个是，`obj.counter`必须有类型`number`，而不是`0`这种固定的文本类型，因为类型要用来同时决定 _读_ 和 _写_ 行为。

这个对字符串类似：

```ts twoslash
// @errors: 2345
declare function handleRequest(url: string, method: "GET" | "POST"): void;
// ---cut---
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
```

在上述例子中`req.method`被推断为`string`类型，不是`"GET"`这种文本类型。因为代码中在赋予特定`req`和调用`handleRequest`之间可能把`req.method`赋予像`"GUESS"`这种值，TypeScript认为这种代码有错误。

有两种方式解决这个问题。

1. 你可以通过在两种位置添加类型断言改变推断内容：

   ```ts twoslash
   declare function handleRequest(url: string, method: "GET" | "POST"): void;
   // ---cut---
   // Change 1:
   const req = { url: "https://example.com", method: "GET" as "GET" };
   // Change 2：
   handleRequest(req.url, req.method as "GET");
   ```
   改变1意味着“我认为`req.method`总会是固定的文本类型`"GET"`”，阻止之后对于这个字段的可能是`GUESS`的猜想。
   改变2意味着“我出于某些原因知道`req.method`有`"GET"`类型”。   

2. 你可以使用`as const`来将整个对象转换为文本类型：

   ```ts twoslash
   declare function handleRequest(url: string, method: "GET" | "POST"): void;
   // ---cut---
   const req = { url: "https://example.com", method: "GET" } as const;
   handleRequest(req.url, req.method);
   ```

`as const`后缀的作用类似于`const`，但是对于类型系统，是确保为所有属性赋予文本类型，而不是更通用的版本例如`string`或`number`

## `null` 和 `undefined`

JavaScript有两种原生类型来表示不存在或未初始化的值：`null`和`undefined`。

TypeScript有两个对应同名的类型（null和undefined）。这两个值的行为取决于你是否开启了[`strictNullChecks`](/tsconfig#strictNullChecks)选项。

### 关闭`strictNullChecks`

关闭了[`strictNullChecks`](/tsconfig#strictNullChecks)选项，可能会是`null`或`undefined`的值可以还被正常获取到，并且`null`和`undefined`的值可以被赋予任何一个类型的属性。
这个类似于一些没有null检查的语言的行为（例如C#, Java）。缺少这些值的检查是造成bug的主要原因之一；我们总是建议人们开启[`strictNullChecks`](/tsconfig#strictNullChecks)，如果在他们代码库中这样做是可行的。


### 开启`strictNullChecks`

开启了[`strictNullChecks`](/tsconfig#strictNullChecks)选项，当一个值是`null`或`undefined`，你将会需要在使用这个值的方法或者属性之前检测这些值。
就像检查在使用可选属性之前检查`undefined`一样，我们可以 _缩小_ 他们来检查可能是`null`的值：

```ts twoslash
function doSomething(x: string | null) {
  if (x === null) {
    // do nothing
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}
```

### 非空断言运算符 (前缀 `!`)

TypeScript也有一些特殊的语法来从一个类型删除`null`和`undefined`（就是不需要上面的if筛除判断了），而不用任何特定的检查。
在一个表达式之后写`!`是一个高效的类型断言指明这个值不是`null`或`undefined`。

```ts twoslash
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
```

就像其他类型断言（就相当于你确定知道这个值，你判断错了ts也不能帮你运行时检查了），这个不会改变你代码的运行时行为，所以要注意只在你很确定值 _不会是_ `null` 或 `undefined`的时候，你才能用 `!`。

## 枚举

枚举是一个通过TypeScript添加到JavaScript的特性，允许你描述一个值，该值可能是一组可能的命名常量之一。不像大多数TypeScript特性，这个 _不是_ 一个给JavaScript编译时类型层级的特性，而是会添加到语言和运行时的。因此，正因为如此，这是一个你应该知道存在的功能，但除非你确定，否则你要延迟使用。你可以读到更多关于枚举的知识在[枚举参考页](/docs/handbook/enums.html)。

## 不太常见的原生类型

值得提一下我们类型系统还有的的JavaScript里剩余的一些原生类型。
我们不会在这里深入讨论。

#### `bigint`

自从ES2020开始，JavaScript中有一个原生类型用于非常大的整数，`BigInt`：

```ts twoslash
// @target: es2020

// Creating a bigint via the BigInt function
const oneHundred: bigint = BigInt(100);

// Creating a BigInt via the literal syntax
const anotherHundred: bigint = 100n;
```

你可以在[TypeScript 3.2版本日志](/docs/handbook/release-notes/typescript-3-2.html#bigint)里学习更多大整数类型。

#### `symbol`

JavaScript里还有一个原生类型用来通过函数`Symbol()`创建全局唯一的引用：

```ts twoslash
// @errors: 2367
const firstName = Symbol("name");
const secondName = Symbol("name");

if (firstName === secondName) {
  // Can't ever happen
}
```

你可以在[Symbols参考页](/docs/handbook/symbols.html)学习他们更多知识。