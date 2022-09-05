---
title: Object Types
layout: docs
permalink: /zh/docs/handbook/2/objects.html
oneline: "How TypeScript describes the shapes of JavaScript objects."
---

在JavaScript里，我们组织和传输数据的基本方法是通过对象。
在TypeScript里，我们通过 _对象类型_ 来实现这些。

正如我们看到的，他们可以是匿名的：

```ts twoslash
function greet(person: { name: string; age: number }) {
  //                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  return "Hello " + person.name;
}
```

或者他们可以通过接口命名

```ts twoslash
interface Person {
  //      ^^^^^^
  name: string;
  age: number;
}

function greet(person: Person) {
  return "Hello " + person.name;
}
```

或者类型别名。

```ts twoslash
type Person = {
  // ^^^^^^
  name: string;
  age: number;
};

function greet(person: Person) {
  return "Hello " + person.name;
}
```

在上面三个例子中，我们已经写了对象包含`name`属性（必须是一个`string` 类型）和`age`属性（必须是一个`number`类型）。

## 属性修饰符

在对象里的每个属性可以指明一系列东西：类型，是否是可选的，和属性是否可以被写入的。

### 可选属性

大多数时候，我们发现处理的对象 _可能_ 会有一个属性。
这种情况，我们可以通过在他们名字后面加一个问号（`?`）将这些属性标记为 _可选的_ 。

```ts twoslash
interface Shape {}
declare function getShape(): Shape;

// ---cut---
interface PaintOptions {
  shape: Shape;
  xPos?: number;
  //  ^
  yPos?: number;
  //  ^
}

function paintShape(opts: PaintOptions) {
  // ...
}

const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
paintShape({ shape, yPos: 100 });
paintShape({ shape, xPos: 100, yPos: 100 });
```

在这个例子里，`xPos`和`yPos`被看做可选的。
我们可以选择是否提供他们两个，所以上面对于`paintShape`的调用方式是有效的。
所有的可选属性真正说明的是，如果属性 _确实_ 要被设置，最好有一个特定的属性。

我们也可以从哪些可选属性获取值（js里正常，会返回undefined） — 但是当我们在[`strictNullChecks`](/tsconfig#strictNullChecks)配置下，TypeScript会告诉我们他们潜在是 `undefined`。
```ts twoslash
interface Shape {}
declare function getShape(): Shape;

interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}

// ---cut---
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos;
  //              ^?
  let yPos = opts.yPos;
  //              ^?
  // ...
}
```

在JavaScript里，即使属性永远没被设置，我们还是可以获取它 — 它只是返回值 `undefined`。
我们只用特殊处理`undefined`就行。

```ts twoslash
interface Shape {}
declare function getShape(): Shape;

interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}

// ---cut---
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos === undefined ? 0 : opts.xPos;
  //  ^?
  let yPos = opts.yPos === undefined ? 0 : opts.yPos;
  //  ^?
  // ...
}
```

注意这种给未指定的值设置默认值太常见了，以至于JavaScript有语法支持他。（下面的 `= 0`）

```ts twoslash
interface Shape {}
declare function getShape(): Shape;

interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}

// ---cut---
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
  console.log("x coordinate at", xPos);
  //                             ^?
  console.log("y coordinate at", yPos);
  //                             ^?
  // ...
}
```

这里我们对`paintShape`的参数使用 [解构模式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)，并且给`xPos`和`yPos`提供[默认值](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Default_values)。
现在`xPos`和`yPos`都在`paintShape`的函数体里被定义，但是对于调用`paintShape`时可选的。

> 注意现在没有方式给解构对象提供类型注释。
> 这是因为下面的语法在JavaScript里意味着不同的事情

```ts twoslash
// @noImplicitAny: false
// @errors: 2552 2304
interface Shape {}
declare function render(x: unknown);
// ---cut---
function draw({ shape: Shape, xPos: number = 100 /*...*/ }) {
  render(shape);
  render(xPos);
}
```

在解构模式下，`shape: Shpae`意味着“从`Shape`获取属性，并且本地定义一个名叫`Shape`的变量”。
同样`xPos: number`会创建一个名叫`number`的变量，他的值基于参数的`xPos`属性。

使用[映射修饰符](/docs/handbook/2/mapped-types.html#mapping-modifiers)，你可以删除 `optional` 属性。

### `readonly` 属性

TypeScript里属性也可以被标记成`readonly`。
当然他不会改变运行时的行为，一个被标记为`readonly`的属性在类型检查时不能被写入。

```ts twoslash
// @errors: 2540
interface SomeType {
  readonly prop: string;
}

function doSomething(obj: SomeType) {
  // We can read from 'obj.prop'.
  console.log(`prop has the value '${obj.prop}'.`);

  // But we can't re-assign it.
  obj.prop = "hello";
}
```

使用`readonly`修饰符不会让一个值完全变成不可修改的 — 或者换句话说，只是他的内部属性不能变。
意味着它的属性是可以被重写的。

```ts twoslash
// @errors: 2540
interface Home {
  readonly resident: { name: string; age: number };
}

function visitForBirthday(home: Home) {
  // We can read and update properties from 'home.resident'.
  console.log(`Happy birthday ${home.resident.name}!`);
  home.resident.age++;
}

function evict(home: Home) {
  // But we can't write to the 'resident' property itself on a 'Home'.
  home.resident = {
    name: "Victor the Evictor",
    age: 42,
  };
}
```

处理`readonly`含义的意外操作很重要。
在TypeScript的开发期间发出关于使用对象的意图很有用。
TypeScript在检查两种类型是否兼容时不会考虑这两种类型的属性是否为`readonly`，因此`readonly`属性也可以通过别名进行修改。

```ts twoslash
interface Person {
  name: string;
  age: number;
}

interface ReadonlyPerson {
  readonly name: string;
  readonly age: number;
}

let writablePerson: Person = {
  name: "Person McPersonface",
  age: 42,
};

// works
let readonlyPerson: ReadonlyPerson = writablePerson;

console.log(readonlyPerson.age); // prints '42'
writablePerson.age++;
console.log(readonlyPerson.age); // prints '43'
```

使用[映射修饰符](/docs/handbook/2/mapped-types.html#mapping-modifiers)，你可以删除 `readonly` 属性。

### 索引签名

有时候你不必提前知道类型属性的所有名字，但是你需要知道类型的形状。

这种情况下，你可以使用索引签名来描述类型可能的值，例如：

```ts twoslash
declare function getStringArray(): StringArray;
// ---cut---
interface StringArray {
  [index: number]: string;
}

const myArray: StringArray = getStringArray();
const secondItem = myArray[1];
//     ^?
```

在上面，我们有一个`StringArray`接口有一个索引签名。
这个索引签名表明当一个`StringArray`被用`number`索引获取的时候，它会返回`string`类型。

一个索引签名属性类型要么是‘string’要么是‘number’。

<details>
    <summary>同时支持两种索引类型是可能的：</summary>
    <p>同时支持两种索引是可能的，但从数字索引返回的类型必须是字符串索引返回的子类型。因为当时用<code>number</code>索引进行索引时，JavaScript实际上会索引到对象之前将其转换为<code>string</code>。这意味使用<code>100</code>索引和使用<code>"100"</code>（一个<code>string</code>）是一样的，所以要保持一致要有上面“数字索引返回的类型必须是字符串索引返回的子类型”的要求。</p>

```ts twoslash
// @errors: 2413
// @strictPropertyInitialization: false
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// Error: indexing with a numeric string might get you a completely separate type of Animal!
interface NotOkay {
  [x: number]: Animal;
  [x: string]: Dog;
}
```

</details>

当我们的字符串索引签名是一个强大的方式描述“字典”模式，他们也会强迫所有的类型匹配他们的返回值类型。
这是因为字符串索引描述`obj.property`和`obj["propety"]`等同的。
在下面的例子里，`name`的类型不会和字符串的索引类型匹配（`[index: string]: number;`的`number`），然后类型检查器报错。

```ts twoslash
// @errors: 2411
// @errors: 2411
interface NumberDictionary {
  [index: string]: number;

  length: number; // ok
  name: string;
}
```

然而，如果索引签名是一个属性类型的联合类型时，属性的不同类型是可以接受的：

```ts twoslash
interface NumberOrStringDictionary {
  [index: string]: number | string;
  length: number; // ok, length is a number
  name: string; // ok, name is a string
}
```

最后，你可以让索引签名是`readonly`的，来防止被索引值被赋值：

```ts twoslash
declare function getReadOnlyStringArray(): ReadonlyStringArray;
// ---cut---
// @errors: 2542
interface ReadonlyStringArray {
  readonly [index: number]: string;
}

let myArray: ReadonlyStringArray = getReadOnlyStringArray();
myArray[2] = "Mallory";
```

你不能设置`myArray[2]`因为索引签名是`readonly`。

## 拓展类型

有的类型可能有多个版本的别的类型是很常见的。
例如，我们可能有一个`BasicAddress`类型描述在美国发送信件和包裹的必要字段。

```ts twoslash
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

在一些情况下这个足够了，但是地址某个地址建筑物有多个单元，则地址通常有一个与之关联的单元号。
然后我们可以描述一个`AddressWithUnit`。

<!-- prettier-ignore -->
```ts twoslash
interface AddressWithUnit {
  name?: string;
  unit: string;
//^^^^^^^^^^^^^
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
```

这个完成了他的工作，但是坏处是我们必须重复所有`BasicAddress`字段，但是我们的改变完全是可增加的。
相反我们可以扩展原始的`BasicAddress`类型然后仅仅给`AddressWithUnit`添加一个新的唯一字段。

```ts twoslash
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}

interface AddressWithUnit extends BasicAddress {
  unit: string;
}
```

在`interface`上的`extends`关键字允许我们有效地从其他命名类型复制成员，并添加我们想要的任何新成员。
这个对于减少我们必须编写的类型声明样板的数量以及表明同一属性的多个不同声明可能相关的意图会很有用。
例如`AddressWithUnit`不需要重复`street`属性，并且因为`street`从`BasicAddress`原始出现，一个读你代码的人将会知道那两个类型在某些方面是相关的。

`interface`也可以从多个类型拓展。

```ts twoslash
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

interface ColorfulCircle extends Colorful, Circle {}

const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
};
```

## 交叉口类型

`inerface`允许我们通过拓展方式来从其他类型创建新的类型。
TypeScript也提供了另一种叫做 _交叉口类型_ 的构造方式，主要用来将两个现有的对象类型组合起来。

一个交叉口类型通过`&`操作符来定义。

```ts twoslash
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}

type ColorfulCircle = Colorful & Circle;
```

这里，我们已经把`Colorful`和`Circle`相交生成了一个新类型，它包含`Colorful` _和_ `Circle`的所有成员。

```ts twoslash
// @errors: 2345
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
// ---cut---
function draw(circle: Colorful & Circle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}

// okay
draw({ color: "blue", radius: 42 });

// oops
draw({ color: "red", raidus: 42 });
```

## Interfaces vs. Intersections

总结上面，我们看到了`interface extend`和`&` 两种方式组合类型比较类似，但是实际上有细微不同。
用接口，我们可以用一个`extends`作用域来从其他类型扩展，并且我们可以用交叉口和类型别名完成类似的事情。
两个原则不同之处是处理冲突的方式，并且这个不同之处是你应该选择interface和交叉口类型的主要原因。

```ts twoslash
interface NumberToStringConverter {
	convert: (value: number) => string;
}

interface BidirectionalStringNumberConverter extends NumberToStringConverter {
	convert: (value: string) => number;
}
```

```ts twoslash
type NumberToStringConverter = {
    convert: (value: number) => string;
}

type BidirectionalStringNumberConverter = NumberToStringConverter & {
    convert: (value: string) => number;
};

// And this is a good thing indeed as a value conforming to the type is easily conceived
const converter: BidirectionalStringNumberConverter = {
    convert: (value: string | number) => {
        return (typeof value === 'string' ? Number(value) : String(value)) as string & number; // type assertion is an unfortunately necessary hack.
    }
}
const s: string = converter.convert(0); // `convert`'s call signature comes from `NumberToStringConverter`

const n: number = converter.convert('a'); // `convert`'s call signature comes from `BidirectionalStringNumberConverter`
```

## 对象类型泛型

让我们假设一个`Box`类型，可以接收任何类型 — `string`，`number`和`Giraffe`等等东西。

```ts twoslash
interface Box {
  contents: any;
}
```

现在`contents`属性是`any`类型，看上去可以工作，但是会变成原始的JavaScript。

我们可以相反使用`unknown`类型，但是这种情况意味着我们已经知道`contents`的类型，我们需要做预防性检查，或者使用能有出错的类型断言。

```ts twoslash
interface Box {
  contents: unknown;
}

let x: Box = {
  contents: "hello world",
};

// we could check 'x.contents'
if (typeof x.contents === "string") {
  console.log(x.contents.toLowerCase());
}

// or we could use a type assertion
console.log((x.contents as string).toLowerCase());
```

一种类型安全的方法是为每种`Box`类型定义`contents`的类型。

```ts twoslash
// @errors: 2322
interface NumberBox {
  contents: number;
}

interface StringBox {
  contents: string;
}

interface BooleanBox {
  contents: boolean;
}
```

但是这个意味着我们将要创建不同的函数，或者重载函数，来对这些不同的类型进行操作。

```ts twoslash
interface NumberBox {
  contents: number;
}

interface StringBox {
  contents: string;
}

interface BooleanBox {
  contents: boolean;
}
// ---cut---
function setContents(box: StringBox, newContents: string): void;
function setContents(box: NumberBox, newContents: number): void;
function setContents(box: BooleanBox, newContents: boolean): void;
function setContents(box: { contents: any }, newContents: any) {
  box.contents = newContents;
}
```

上面代码需要很多样板，此外，我们可能之后需要增加新的类型和更多重载。
这个让人筋疲力尽，因为我们的盒子类型和重载实际上都是相同的。

相反，我们可以让`Box`变成泛型，然后定义 _类型参数_。

```ts twoslash
interface Box<Type> {
  contents: Type;
}
```

你可能要把这个解读成：“一个`Type`的`Box`是一个对象，他的`contents`拥有类型`Type`”。
之后，当我们引用`Box`的时候，我们需要在`Type`的位置给一个 _类型参数_。

```ts twoslash
interface Box<Type> {
  contents: Type;
}
// ---cut---
let box: Box<string>;
```

想象`Box`是一个真实类型模板，而`Type`是一个占位符，之后会被其他类型替代。
当TypeScript看见`Box<string>`的时候，它会将`Box<Type>`里的每个`Type`实例替换，并且最终变成`{ contents: string }`的样子。
换句话说，`Box<string>`和我们更早定义的`StringBox`工作方式一致。

```ts twoslash
interface Box<Type> {
  contents: Type;
}
interface StringBox {
  contents: string;
}

let boxA: Box<string> = { contents: "hello" };
boxA.contents;
//   ^?

let boxB: StringBox = { contents: "world" };
boxB.contents;
//   ^?
```

`Box`是可以复用的，因为`Type`可以传入任何类型。意味着当我们需要一个新类型的盒子，我们不需要定义一个新的`Box`类型（尽管如果我们愿意，我们当然也可以）。

```ts twoslash
interface Box<Type> {
  contents: Type;
}

interface Apple {
  // ....
}

// Same as '{ contents: Apple }'.
type AppleBox = Box<Apple>;
```

这个也意味着我们通过使用[泛型函数](/docs/handbook/2/functions.html#generic-functions)完全可以避免重载。

```ts twoslash
interface Box<Type> {
  contents: Type;
}

// ---cut---
function setContents<Type>(box: Box<Type>, newContents: Type) {
  box.contents = newContents;
}
```

值得注意的是，类型别名也可以是泛型的。我们可以定义新的`Box<Type>`接口，以前是：

```ts twoslash
interface Box<Type> {
  contents: Type;
}
```

使用类型别名是：

```ts twoslash
type Box<Type> = {
  contents: Type;
};
```

因为类型别名不像接口，可以描述不止于对象类型的东西，我们也可以使用他们来写其他类型的泛型辅助类型。

```ts twoslash
// @errors: 2575
type OrNull<Type> = Type | null;

type OneOrMany<Type> = Type | Type[];

type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>;
//   ^?

type OneOrManyOrNullStrings = OneOrManyOrNull<string>;
//   ^?
```
1
稍后我们将回过头讨论类型别名，

###  `Array` 类型

泛型对象类型通常是某种容器类型，它们独立于它们所包含的元素类型工作。
数据结构以这种方式工作是理想的，这样它们就可以在不同的数据类型中复用。

事实上，我们一直在用与本手册中类似的类型：`Array`类型。
当我们写像`number[]`或`string[]`类型的时候，那也是`Array<number>`和`Array<string>`的简写。

```ts twoslash
function doSomething(value: Array<string>) {
  // ...
}

let myArray: string[] = ["hello", "world"];

// either of these work!
doSomething(myArray);
doSomething(new Array("hello", "world"));
```

比较像上面的`Box`类型，`Array`本身也是泛型类型。

```ts twoslash
// @noLib: true
interface Number {}
interface String {}
interface Boolean {}
interface Symbol {}
// ---cut---
interface Array<Type> {
  /**
   * Gets or sets the length of the array.
   */
  length: number;

  /**
   * Removes the last element from an array and returns it.
   */
  pop(): Type | undefined;

  /**
   * Appends new elements to an array, and returns the new length of the array.
   */
  push(...items: Type[]): number;

  // ...
}
```

现代JavaScript也提供了其他是泛型的数据结构，像`Map<K, V>, Set<T>`，和`Promise<T>`。
所有这一切真正意味着由于 `Map`、`Set` 和 `Promise` 的行为方式，它们可以与任何类型的集合一起工作。

### `ReadonlyArray` 类型

`ReadonlyArray`是一个特殊的类型，描述数组不应该被改变。

```ts twoslash
// @errors: 2339
function doStuff(values: ReadonlyArray<string>) {
  // We can read from 'values'...
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);

  // ...but we can't mutate 'values'.
  values.push("hello!");
}
```

比较像属性的`readonly`修饰器，他主要是用于意图的工具。（只是ts会报错，但是js没有具体实现，表明意图。）
当我们看见一个函数返回`ReadonlyArray`的时候，它告诉我们不应该改变内容，并且当我们看见一个函数需要一个`ReadonlyArray`参数，它告诉我们可以给函数传递任何值不用担心它会改变参数的内容。

不像`Array`，`ReadonlyArray`结构体没有可以使用的构造函数。

```ts twoslash
// @errors: 2693
new ReadonlyArray("red", "green", "blue");
```

然而我们可以给`ReadonlyArray`赋予普通的`Array`值。

```ts twoslash
const roArray: ReadonlyArray<string> = ["red", "green", "blue"];
```

就像TypeScript给`Array<Type>`提供`Type[]`的简写语法，它也它也给`ReadonlyArray<Type>`提供`readonly Type[]`的简写语法。

```ts twoslash
// @errors: 2339
function doStuff(values: readonly string[]) {
  //                     ^^^^^^^^^^^^^^^^^
  // We can read from 'values'...
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);

  // ...but we can't mutate 'values'.
  values.push("hello!");
}
```

最后一个值得注意的事情是不像`readonly`属性修饰器（在哪里讲过？★），可分配性在常规 `Array`s 和 `ReadonlyArray`s 之间不是双向的。

```ts twoslash
// @errors: 4104
let x: readonly string[] = [];
let y: string[] = [];

x = y;
y = x;
```

### 元组类型

一个元组类型是另一种`Array`类型，它可以准确知道它包含哪些类型，并且在特定位置是什么特定类型。

```ts twoslash
type StringNumberPair = [string, number];
//                      ^^^^^^^^^^^^^^^^
```

这里`StringNumberPair`是一个`string`和`number`的元组类型。
就像`Readon1
```

果我们视图索引超过元素的数量，我们会得到一个错误。

```ts twoslash
// @errors: 2493
function doSomething(pair: [string, number]) {
  // ...

  const c = pair[2];
}
```

我们可以使用JavaScript数组结构来[解构元组](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Array_destructuring)。

```ts twoslash
function doSomething(stringHash: [string, number]) {
  const [inputString, hash] = stringHash;

  console.log(inputString);
  //          ^?

  console.log(hash);
  //          ^?
}
```

> 元组类型在大量基于约定的API中很有用，其中每个元素的含义都是“显而易见的”。
> 这个让我们在解构变量时可以灵活地命名变量。
> 在上面的例子里，我们能够将元素`0`和`1`命名为我们想要的任何内容。
>
> 然而，因为不是每个用户能够对显而易见的事物有相同的看法，因此可能值得重新考虑使用具有描述性的对象可能对你的API更合适。

除了上面例子的长度检查，下面这样简单元组类型等价于为特定索引声明的`Array`版本的类型，以及可以使用文本数字类型定义`langth`的值。（下面这种interface写法和元组类型一致）
```ts twoslash
interface StringNumberPair {
  // specialized properties
  length: 2;
  0: string;
  1: number;

  // Other 'Array<string | number>' members...
  slice(start?: number, end?: number): Array<string | number>;
}
```

另一件你可能感兴趣的事情是元组可以有可选属性，通过在类型最后写一个`?`实现。
可选的元组属性只能在最后出现，并且会影响元组的`length`。

```ts twoslash
type Either2dOr3d = [number, number, number?];

function setCoordinate(coord: Either2dOr3d) {
  const [x, y, z] = coord;
  //           ^?

  console.log(`Provided coordinates had ${coord.length} dimensions`);
  //                                            ^?
}
```

元组也可以有扩展元素，但他们必须是数组/元组类型。

```ts twoslash
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
```

- `StringNumberBooleans`描述了一个元组，前两个元素分别是`string`和`number`，但是后面可能跟着任意数量的`boolean`类型。
- `StringBooleansNumber`描述了一个元组，第一个参数是`string`然后跟着任意数量的`boolean`，然后结尾是`number`。
- `BooleansStringNumber`描述了一个元组，开始的参数是任意数量的`booleans`，然后结尾是一个`string`和一个`number`。

一个有扩展元素的元组类型没有`length`属性 — 它只在不同位置有一组知名元素。

```ts twoslash
type StringNumberBooleans = [string, number, ...boolean[]];
// ---cut---
const a: StringNumberBooleans = ["hello", 1];
const b: StringNumberBooleans = ["beautiful", 2, true];
const c: StringNumberBooleans = ["world", 3, true, false, true, false, true];
```

为什么可选和扩展元素很有用？
嗯，它允许TypeScript将元组与参数列表对应起来。
元组类型可以用在[剩余参数和扩展运算符里](/docs/handbook/2/functions.html#rest-parameters-and-arguments)，所有能像下面例子这样使用：

```ts twoslash
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args;
  // ...
}
```

基本等同于：

```ts twoslash
function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
```

当你想用一个剩余参数获取可变数量的参数时，这很方便，并且你需要最少数量的元素，并且不用引入中间变量。

<!--
TODO do we need this example?

For example, imagine we need to write a function that adds up `number`s based on arguments that get passed in.

```ts twoslash
function sum(...args: number[]) {
    // ...
}
```

We might feel like it makes little sense to take any fewer than 2 elements, so we want to require callers to provide at least 2 arguments.
A first attempt might be

```ts twoslash
function foo(a: number, b: number, ...args: number[]) {
    args.unshift(a, b);

    let result = 0;
    for (const value of args) {
        result += value;
    }
    return result;
}
```

-->

### `readonly` 元组类型

元组类型的最后一点是 — 元祖类型有`readonly`变量，并且可以通过在前面加上`readonly`修饰符来制定 — 就像数组简写语法一样。

```ts twoslash
function doSomething(pair: readonly [string, number]) {
  //                       ^^^^^^^^^^^^^^^^^^^^^^^^^
  // ...
}
```

正如你期待的，给任意一个`readonly`的元组值写入值在TypeScript里是不被允许的。

```ts twoslash
// @errors: 2540
function doSomething(pair: readonly [string, number]) {
  pair[0] = "hello!";
}
```

在大多数代码中，元组往往被创建并且未修改，因此尽可能将类型注释为`只读`元组是一个很好的默认值。
下面一点也很重要，带有`const`断言的数组字面量将使用`readonly`元组类型进行推断。（就是用了`const`断言之后数组字面量被转换为`readonly`的元组类型）

```ts twoslash
// @errors: 2345
let point = [3, 4] as const;

function distanceFromOrigin([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2);
}

distanceFromOrigin(point);
```

这里`distanceFromOrigin`永远不会修改他的入参，但是希望传入一个可修改的元组。
因为`point`的类型从`readonly [3, 4]`推断，它不能和`[number, number]`匹配，因为这个类型不能保证`point`的元素不会被修改。

<!-- ## Other Kinds of Object Members

Most of the declarations in object types:

### Method Syntax

### Call Signatures

### Construct Signatures

### Index Signatures -->
