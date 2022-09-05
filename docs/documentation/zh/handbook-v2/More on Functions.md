---
title: More on Functions
layout: docs
permalink: /zh/docs/handbook/2/functions.html
oneline: "Learn about how Functions work in TypeScript."
---

函数是任何应用最基本的构建部分，无论它们是本地函数、从其他模组导入的函数，还是类的方法。
他们也是值，就行其他普通值，TypeScript有很多方式描述一个函数怎么被调用。
让我们学习如果写类型来描述函数。

## 函数类型表达式

描述函数最简单的方式是用 _函数类型表达式_。
这种表达式对于箭头函数也是语法上类似的：

```ts twoslash
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}

function printToConsole(s: string) {
  console.log(s);
}

greeter(printToConsole);
```

语法`(a:string) => void` 意味着一个函数有一个参数，叫做`a`，是字符串类型，并且函数没有返回值。
就像函数声明一样，如果一个参数类型没有被指明，它隐式是`any`类型。（声明一个函数，没有给输入输出信息默认是`any`）。

> 注意参数名是**必须的**。函数类型`(string) => void`意味着"一个函数有一个`any`类型的`string`"！

当然我们可以使用类银杏谷别名来定义一个函数类型：（类型别名定义入参和返回值）

```ts twoslash
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
  // ...
}
```

## 调用签名

在JavaScript里，函数可以有些属性被调用时使用。
但是上面讲的定义函数类型的语法不允许描述更多属性值。
如果我们想要描述一些可以在调用时使用的属性，我们可以在对象类型里写一些 _调用签名_ ：

```ts twoslash
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
```

注意这个语法和函数类型语法的类型不同 — 在参数列表和返回值之间用`:`而不是`=>`。

## 构造签名

JavaScript函数也可以用`new`操作符被调用。
TypeScript会把用`new`操作符调用的看做 _constructors构造函数_，因为他们通常会创建新对象。
你可以通过在调用签名前面加一个`new`关键字写一个 _构造签名_：

```ts twoslash
type SomeObject = any;
// ---cut---
type SomeConstructor = {
  new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

有些对象，例如JavaScript的`Date`对象，可以用`new`或者不用`new`调用。（★这里有eventBus用原型链定义，export的时候new报错的问题可以探讨）
你可以在同一类型声明组合调用签名和构造签名：（上面这里写很多都是对于一个变量或者参数定义为函数的写法，但是自己用function或者箭头函数写不上去，除非用as 类型断言？）

```ts twoslash
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}
```

## 泛型函数

写一个函数的返回值类型和输入值类型相关，或者两个输入参数以某种方式相关很常见。
让我们看下面的函数返回一个数组的第一个参数：

```ts twoslash
function firstElement(arr: any[]) {
  return arr[0];
}
```

这个函数完成了他的工作，但是不信的是返回值类型是`any`。
最好能我们获取数组元素的类型。

在Typescript里，_泛型_ 是我们想要描述两个值之间的对应关系时用到的。
我们通过在函数签名声明一个 _类型参数_ 实现：

```ts twoslash
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}
```

通过在这个函数添加类型参数`Type`，并且在两个地方使用，我们就在函数的输入（数组）和输出（返回值）之间建立了联系。
现在当我们调用的时候，一个更精确的类型出现了：

```ts twoslash
declare function firstElement<Type>(arr: Type[]): Type | undefined;
// ---cut---
// s is of type 'string'
const s = firstElement(["a", "b", "c"]);
// n is of type 'number'
const n = firstElement([1, 2, 3]);
// u is of type undefined
const u = firstElement([]);
```

### 推断

注意我们在这个李自力没有指明`Type`是什么。
类型是被TypeScript里被TypeScript _推断_ 出来的。

我们也可以使用多个类型参数。
例如，一个`map`的独立TypeScript版本应该如下所示：

```ts twoslash
// prettier-ignore
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}

// Parameter 'n' is of type 'string'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```

注意在我们的例子里，TypeScript可以同时推断出`Input`和`Output`类型的具体类型。（`Input`从输入的`string`数组，`Output`从函数表达式的`parseInt`返回值（`number`））

### 约束

我们已经在对 _any_ 类型参数应用了泛型函数。
有时候我们想要关联两个值，但是要求他们要有共同特定的子集。
这种情况下，我们可以使用 _约束_ 来限制类型的种类，告诉我们的类型参数可以接受怎么样的参数。

让我们写一个比较两个值长度的函数。
为了实现这个，我们要求值的`length`返回一个数字。
我们通过`extends` 子句来 _约束_ 参数类型：

```ts twoslash
// @errors: 2345 2322
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}

// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'alice' | 'bob'
const longerString = longest("alice", "bob");
// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
```

在上面例子里有几点有趣的需要强调。
我们允许TypeScript 从`longest`的返回类型 _推断_。
返回值的推断结果也会在泛型函数起作用。（具体意思看下面）

因为我们限制`Type`是`{length: number}`样子，我们允许获取`a`和`b`参数的`.length`属性。
如果没有写类型约束（`{length: number}`），我们不能获取length属性值，因为值可能是其他类型没有length属性。

`longerAray`和`longerString`的返回值基于参数推断出来。
请记住，泛型用来将两个或者多个值用相同类型联系起来！（这里是利用推断把输入值和输出值联系起来了）

最后正如我们猜想的，调用`longest(10, 100)`被拒绝了，因为`number`类型乜有`.length`属性。

### 使用被约束的值

下面是我们使用泛型约束时常见的错误

```ts twoslash
// @errors: 2322
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum };
  }
}
```

看上去函数时ok的 — `Type`被约束成`{length: number}`，并且函数要么返回`Type`要么返回一个“满足约束样子的值”。
问题就是函数定义时要求返回一个 _相同_ 类型的对象，而不是 _哪些_ 样子匹配约束规则的对象。
如果和这个代码不报错，你可能之后写的代码不能正常工作（下面返回只有length的，slice就报错了）：

```ts twoslash
declare function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type;
// ---cut---
// 'arr' gets value { length: 6 }
const arr = minimumLength([1, 2, 3], 6);
// and crashes here because arrays have
// a 'slice' method, but not the returned object!
console.log(arr.slice(0));
```

### 指定类型参数

TypeScript通常可以从从泛型调用里推断出理想的值，但是特殊情况会有问题。
看底下例子，假设你要写一个函数合并两个数组：

```ts twoslash
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}
```

用两个不一致的函数调用的时候通常会报错（combine函数约束要都是Type，两个不一致）：

```ts twoslash
// @errors: 2322
declare function combine<Type>(arr1: Type[], arr2: Type[]): Type[];
// ---cut---
const arr = combine([1, 2, 3], ["hello"]);
```

如果你想要实现这个用法，你可以手动指定`Type`可能的值：

```ts twoslash
declare function combine<Type>(arr1: Type[], arr2: Type[]): Type[];
// ---cut---
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```

### 编写良好泛型函数的指南

写泛型函数很有趣，并且它可能很容易被类型参数迷惑。
写太多类型参数或者写不必要的约束，可能会让推理戳粗，使你的函数调用者心烦意乱。（因此下面有些指南）

#### 将类型参数下移

这里有个函数有两种相似的写法：

```ts twoslash
function firstElement1<Type>(arr: Type[]) {
  return arr[0];
}

function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0];
}

// a: number (good)
const a = firstElement1([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]);
```

这两种方法第一眼看上去可能一样，但是`firstElement1`是写这个函数的更好方法。
因为他的推断返回值是`Type`，但是`firstElement2`的推断返回值是`any`，因为TypeScript必须使用约束类型（`extneds any[]`这一部分，编写代码时判断）来判断`arr[0]`的表达式，而不是“等”到函数调用的时候动态判断出元素类型。
These might seem identical at first glance, but `firstElement1` is a much better way to write this function.

> **规则**：尽可能的单独使用类型参数，减少他添加约束

#### 使用更少的类型参数

这个是另外一对相似的函数：

```ts twoslash
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
  return arr.filter(func);
}

function filter2<Type, Func extends (arg: Type) => boolean>(
  arr: Type[],
  func: Func
): Type[] {
  return arr.filter(func);
}
```

我们已经创造了一个类型参数`Func` — 它 _不会让任何两个值相关联_。
这是一个红色警告，因为`filter2`这个函数的写法意思是调用者如果想指定类型参数必须手动传入额外不必要的类型参数。（就是一旦写`filter2<>`，尖括号必须传入两个参数，第二个参数没用）
`Func`没有做任何事情，但是让函数能难读和难用了。

> **规则**：总是尽量少使用类型参数

#### 类型参数应该出现两次

有时候我们忘记一个函数可能不需要泛型：

```ts twoslash
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}

greet("world");
```

我们可能只需要写一个更简洁版本：

```ts twoslash
function greet(s: string) {
  console.log("Hello, " + s);
}
```

记住，类型参数是用来 _连接多个值的类型_ 。
如果一个类型参数只是使用了一次，强烈建议考虑你是否需要他。

> **规则**：如果一个类型参数只在一个位置出现一次，强烈建议考虑你是否需要他。

## 可选参数

JavaScript的函数进程传入可选数量的参数。
例如`number`的`toFixed`方法有一个可选的数字参数：

```ts twoslash
function f(n: number) {
  console.log(n.toFixed()); // 0 arguments
  console.log(n.toFixed(3)); // 1 argument
}
```

我们可以在TypeScript里实现，通过用`?`将参数标记为 _可选的_ ：

```ts twoslash
function f(x?: number) {
  // ...
}
f(); // OK
f(10); // OK
```

尽管参数被标记为`number`类型，`x`蚕食实际上有`number | undefined`类型，因为在JavaScript里没有传入和这个参数，会设置这个值为`undefined`。

你也可以提供一个 _默认_ 参数：

```ts twoslash
function f(x = 10) {
  // ...
}
```

现在在`f`的主体里，`x`将会有`number`类型，因为任何`undfined`参数会被替代成`10`。
注意当一个参数是可选的，调用者可以总是传入`undefined`，因为这个参数仅仅代表“缺失”的参数。
```ts twoslash
declare function f(x?: number): void;
// cut
// All OK
f();
f(10);
f(undefined);
```

### 在回调函数里的可选参数

一旦你学过了可选参数和函数类型表达式，当你写有回调的函数时候很容易犯下面的错误：

```ts twoslash
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}
```

人们通常的意思是，把`index?`当做一个可选参数，然后让下面两个调用方式都合法：

```ts twoslash
// @errors: 2532
declare function myForEach(
  arr: any[],
  callback: (arg: any, index?: number) => void
): void;
// ---cut---
myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```

这个 _实际意义_ 是`callback` _可能被一个参数调用_。
换句话说，别人的函数的实现是可能像下面这样：

```ts twoslash
// @errors: 2532
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // I don't feel like providing the index today
    callback(arr[i]);
  }
}
```

反过来，TypeScript 将强制执行此含义并发出实际上不可能的错误：
（就是人们的意思是设置回调函数可选参数（ts的写法含义），想的很美是可以两种方法使用，但是ts的实际意义是`callback`可能被一个函数调用，如果函数体里`callback`有地方用一个参数调用可能会出错）（就是两种意义不一样）

<!-- prettier-ignore -->
```ts twoslash
// @errors: 2532
declare function myForEach(
  arr: any[],
  callback: (arg: any, index?: number) => void
): void;
// ---cut---
myForEach([1, 2, 3], (a, i) => {
  console.log(i.toFixed());
});
```

在JavaScript里，如果你传入比参数更多的参数，额外的参数会忽略。
TypeScript也是这样。
拥有更少参数的函数调用总可以替代参数更多的函数调用（相同函数类型）。

> 当给回调函数写函数类型时，_永远不要_ 写可选参数，除非想要 _调用_ 这个函数时不传入可选参数。（回调函数参数一般在函数体的调用方法是固定死的，除非是函数体里有if else两种传入方式）

## 函数重载

有些JavaScript函数可以被不同数量和类型的参数调用。
例如，你可能想写一个函数返回`Date`，要么传入时间戳（一个参数），要么传入特定年月日（三个参数）。

在TypeScript里，我们通过 _重载签名_ 可以定义一个被不同方式调用的函数。
为了实现这个，先写几个函数声明（通常大于等于两个），紧跟着函数体：

```ts twoslash
// @errors: 2575
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
```

这个例子里，我们写了两个重载：一个接收一个参数，另一个接收三个参数。
前面这两个被称为 _重载签名_。

然后我们编写了一个具有兼容签名的函数实现。
这个函数拥有 _实现签名_，但是这个签名不能直接被调用。
及时我们写一个函数，在必选参数后面带有带有两个可选参数，他也不能被**两个参数**调用！（ts层面报错）。

### 重载签名和实现签名

这是一个常见的混淆情况。
人们经常会写下面样子的代码，并且不理解为什么有错误：

```ts twoslash
// @errors: 2554
function fn(x: string): void;
function fn() {
  // ...
}
// Expected to be able to call with zero arguments
fn();
```

再次，用来写函数体的签名不能被外面“看见”。

> _实现_ 的签名体不能从外面看见。
> 当你写重载函数的时候，你应该总是在函数实现上写 _两个_ 或更多签名。

实现签名必须和重载签名 _匹配_ 。（就是实现要完成所有重载情况，并且要匹配）
例如，下面的函数报错，因为实现签名和重载没有正确匹配：

```ts twoslash
// @errors: 2394
function fn(x: boolean): void;
// Argument type isn't right
function fn(x: string): void;
function fn(x: boolean) {}
```

```ts twoslash
// @errors: 2394
function fn(x: string): string;
// Return type isn't right
function fn(x: number): boolean;
function fn(x: string | number) {
  return "oops";
}
```

### 写好的重载

就像泛型一样，这里也有一些你在使用函数重载的时候应该遵循的指南。
遵循这些指南会让你的函数调用更简单，更简单理解和更简单实现。

让我们考虑一个返回字符串或者数组长度的字符串：

```ts twoslash
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
  return x.length;
}
```

这个函数看起来不错，我们可以传入字符串或数组调用。
然而，我们可能用一个可能是字符串 _或_ 数组的值调用，因为TypeScript一次只能将函数判断为一个重载：

```ts twoslash
// @errors: 2769
declare function len(s: string): number;
declare function len(arr: any[]): number;
// ---cut---
len(""); // OK
len([0]); // OK
len(Math.random() > 0.5 ? "hello" : [0]);
```

因为两种重载有相同的参数个数和相同的返回类型，我们反而可以写一个函数的非重载版本：

```ts twoslash
function len(x: any[] | string) {
  return x.length;
}
```

这个好多了！
调用者可以用任何一种值来调用它，作为额外的好处，我们不必找出正确的实现签名。

> 尽可能的对参数使用联合类型而不是重载。

### 在函数里声明`this`

TypeScript在函数里会通过代码流分析判断`this`应该是什么，例如下面的代码：

```ts twoslash
const user = {
  id: 123,

  admin: false,
  becomeAdmin: function () {
    this.admin = true;
  },
};
```

TypeScript理解了函数`user.becomeAdmin`有一个对应的`this`指的是外层对象的`user`。默认的`this`对于大多数情况足够了，但是有一些情况你需要对`this`代表的含义有更好的控制。JavaScript规范规定你不能有一个不能有一个叫做`this`的变量，因此TypeScript也用这种语法空间让你在函数体里指明`this`的类型。（js里不能声明，所以ts有机会让你来写this的具体类型）。

```ts twoslash
interface User {
  id: number;
  admin: boolean;
}
declare const getDB: () => DB;
// ---cut---
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});
```

这个模式通常用作回调风格的API，在调用函数时通常由另一个对象控制。注意你需要使用`function`而不能用箭头函数才能得到这个行为：（箭头函数的`this`指向全局变量）

```ts twoslash
// @errors: 7041 7017
interface User {
  id: number;
  isAdmin: boolean;
}
declare const getDB: () => DB;
// ---cut---
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(() => this.admin);
```

## 需要知道的其他类型

当你使用函数类型的时候，有一些额外的类型需要你来区分。
与所有类型一样，你可以在任何地方使用他们，但是他们在函数的上下文中很相关。

### `void`

`void`代表一个不会有返回值的函数的返回值。
当一个函数没有任何`return`语句或者从那些返回语句中不返回任何显式值的时候会被推断出来：

```ts twoslash
// The inferred return type is void
function noop() {
  return;
}
```

在JavaScript里，一个没有返回值的函数会隐式返回值`undefined`。
但是`void`和`undefined`在TypeScript里不是同一个东西。
在这章的末尾有很多具体细节。

> `void`和`undefined`不同。

### `object`

特殊类型`object`指的是不是原生类型（ `string`, `number`, `bigint`, `boolean`, `symbol`, `null`, 或 `undefined`）的任何类型。
这个和空对象类型 `{ }` 不同，并且也和全局类型`Object`不同。
你可能永远不会用到`Object`。

> `object`不是`Object`。永远使用`object`！

注意在JavaScript里，函数值是对象：他们有属性，有`Object.prototye`在他们的原型链里，并且是`instanceof Object`，你可以在他们上面调用`Object.keys`等等。
因为这个原因，函数在TypeScript里，函数类型被认为是`object`类型。

### `unknown`

`unknown`类型代表 _任何_ 值。
这个和`any`类型类似，但是`any`比它更安全，因为操作一个`unknown`值是非法的：

```ts twoslash
// @errors: 2571
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b();
}
```

这个当你描述函数类型的时候很有用，因为你可以描述一个函数值可以接受任何类型的值，而不用在函数体里写`any`类型的值。

反过来说，你可以可以定义一个函数返回一个unkonwn类型的值：

```ts twoslash
declare const someRandomString: string;
// ---cut---
function safeParse(s: string): unknown {
  return JSON.parse(s);
}

// Need to be careful with 'obj'!
const obj = safeParse(someRandomString);
```

### `never`

有些函数 _永远不会_ 返回值：

```ts twoslash
function fail(msg: string): never {
  throw new Error(msg);
}
```

`never`类型代表值 _永远不会_ 被观察到的值。
作为一个返回类型里，这个意味着这个函数会抛出异常或者终止项目的执行。

`never`也在TypeScript发现联合类型里没有剩余的值时出现。

```ts twoslash
function fn(x: string | number) {
  if (typeof x === "string") {
    // do something
  } else if (typeof x === "number") {
    // do something else
  } else {
    x; // has type 'never'!
  }
}
```

### `Function`

全局类型`Function`描述了一些像 `bind`, `call`, `apply`，和其他在JavaScript函数值里的一些属性。
它也有一些特殊的属性，说明`Function`类型可以总被调用；这些调用返回`any`类型：

```ts twoslash
function doSomething(f: Function) {
  return f(1, 2, 3);
}
```

这个也叫作 _无类型的函数调用_，并且最好避免，因为有`any`的返回类型。

如果你需要接受一个文本函数并且不打算调用，类型`()=>void`会更安全。

## 剩余参数和扩展运算符

<blockquote class='bg-reading'>
   <p>背景阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters'>Rest Parameters</a><br/>
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax'>Spread Syntax</a><br/>
   </p>
</blockquote>

### Rest Parameters 剩余参数

为了使用可选参数或者重载让函数可以接受一系列固定参数值，我们也可以通过 _剩余参数_ 定义一个接收 _无限制_ 数量参数。

一个剩余参数在所有参数最后出现，使用`...`语法：

```ts twoslash
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```

在TypeScript里，这些参数上的类型声明隐式是`any[]`而不是`any`，并且想要覆盖的话，给定的任何乐行注释必须是`Array<T>`或`T[]`，或者一个元组类型（之后会详细讲解）。

### Rest Arguments 扩展运算符

相反的，我们可以用扩展运算符从一个数组 _提供_ 任意数量的参数。
例如，数组的`push`方法可以传入任何数量的参数：

```ts twoslash
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
```

注意通常来说，TypeScript没有不会认为数组是不可变的。（就是认为数组是可变的，下面ts判断args是动态可变的，可能运行一段代码后两个参数变成三个参数）
这种行为可能导致一些意外的行为：

```ts twoslash
// @errors: 2556
// Inferred type is number[] -- "an array with zero or more numbers",
// not specifically two numbers
const args = [8, 5];
const angle = Math.atan2(...args);
```

对这种情况最好的修复方案取决于你的代码，但是总体一个`const`上下文是最直观的解决方案：

```ts twoslash
// Inferred as 2-length tuple
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```

针对较旧的运行时，使用剩余参数可能需要打开 [`downlevelIteration`](/tsconfig#downlevelIteration)。（开启这个后编译成js会自动进行一些更通用的降级转换）

<!-- TODO link to downlevel iteration -->

## 参数解构

<blockquote class='bg-reading'>
   <p>背景阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment'>Destructuring Assignment</a><br/>
   </p>
</blockquote>

你可以使用参数解构来方便的将提供的参对象在函数体里拆包成一个或几个本地变量。
在JavaScript里，它看起来像下面这样。

```js
function sum({ a, b, c }) {
  console.log(a + b + c);
}
sum({ a: 10, b: 3, c: 9 });
```

对象的类型注释在解构语法之后写：

```ts twoslash
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
```

这看起来有点冗长，但你也可以在此处使用类型别名：

```ts twoslash
// Same as prior example
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
  console.log(a + b + c);
}
```

## 函数的可分配性

### 返回类型`void`

返回值是`void`的函数可以提供一些不寻常，但是期望的行为。
The `void` return type for functions can produce some unusual, but expected behavior.

语义上定义一个`void`返回值**不会**强制函数**不返回值**。另一个方式表达意思是，定义一个上下文函数的返回类型是`void`，当实现了，你可以返回 _任何类型_ 的值，但是会被忽略。

因此，下面对于类型`()=>void`的实现都是有效的：

```ts twoslash
type voidFunc = () => void;

const f1: voidFunc = () => {
  return true;
};

const f2: voidFunc = () => true;

const f3: voidFunc = function () {
  return true;
};
```

并且当这些函数中的一个返回值赋予其他变量的时候，这些值也会保持`void`类型（本意是这个值不会有返回值，ts不会强制，可以被忽略，但你要操作void可能有问题）：

```ts twoslash
type voidFunc = () => void;

const f1: voidFunc = () => {
  return true;
};

const f2: voidFunc = () => true;

const f3: voidFunc = function () {
  return true;
};
// ---cut---
const v1 = f1();

const v2 = f2();

const v3 = f3();
```

因为这个行为，下面代码有效
就是`Array.prototype.push`是`()=>number`类型，并且`Array.prototype.forEach`方法期待是`()=>void`类型，因为被忽略，是有效的。

```ts twoslash
const src = [1, 2, 3];
const dst = [0];

src.forEach((el) => dst.push(el));
```

有一点要特别注意的，但一个“文本函数定义”（就是用function语法显式声明，有入参，返回值，看对比上面的voidFunc先定义的方法）有一个`void`返回值，这个函数一定**不能**返回任何值。

```ts twoslash
function f2(): void {
  // @ts-expect-error
  return true;
}

const f3 = function (): void {
  // @ts-expect-error
  return true;
};
```

关于更多`void`请参考下面其他文档入口：

- [v1 handbook](https://www.typescriptlang.org/docs/handbook/basic-types.html#void)
- [v2 handbook](https://www.typescriptlang.org/docs/handbook/2/functions.html#void)
- [FAQ - "Why are functions returning non-void assignable to function returning void?"](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-functions-returning-non-void-assignable-to-function-returning-void)
