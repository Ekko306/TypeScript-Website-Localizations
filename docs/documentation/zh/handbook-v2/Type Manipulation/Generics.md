---
title: Generics
layout: docs
permalink: /zh/docs/handbook/2/generics.html
oneline: Types which take parameters
---

软件工程的一个重要部分是对于构建一个组件，不仅要有良好定义明确和一致的API，而且还要具有可复用性。
哪些根据今天的数据能够正常工作的组件如果铭泰还能使用将会给你构建大型软件系统的灵活能力。

在哪些像C#和Java的语言里，创建可复用组件的工具箱中一个重要工具是 _泛型_，也就是说，能够创建一个可以在多种类型而不是单一类型上工作的组件。
这样允许用户消费这些组件并且使用他们本身的类型。

## Hello World of Generics

首先，让我们做一下泛型的“hello world”：一个恒等函数。
恒等函数的定义是会返回无论传入的什么值。
你可以把这个想象成类似`echo`命令。

没有泛型，我们要给恒等函数传入一个特定类型：

```ts twoslash
function identity(arg: number): number {
  return arg;
}
```

或者，我们可以通过`any`类型描述一个恒等函数：

```ts twoslash
function identity(arg: any): any {
  return arg;
}
```

虽然使用`any`肯定是通用的，但是它会导致函数允许接收任何类型的`arg`参数，我们实际上丢失了关于类型和函数返回值类型的信息。
如果我们传入一个数字，我们唯一的得到的信息是我们的返回值会是任意类型。

相反，我们需要一个方式来捕获参数类型，然后我们可以使用它表示返回值的类型。
这里，我们会使用 _类型变量_，一个变量的特殊种类，可以在类型上而不是在值上使用。

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}
```

我们现在已经给恒等函数添加了类型变量`Type`。
这个`Type`允许我们从用户提供的类型获取值（例如`number`），所以我们可以在得到信息后使用。
这里我们再次使用`Type`作为返回值。通过检查，我们现在可以看到参数和返回类型使用了相同的类型。
这个允许我们在函数的一侧和另一侧传输该类型的信息。

我们可以说这个版本的`identity`函数是泛型的，因为他对于广泛的类型都能工作。
不像使用`any`，他也与使用数字作为参数和返回类型的`identity`函数一样精确（即，它不会丢失任何信息）。

一旦我们已经写了泛型恒等函数，我们可以两种方式调用。
第一种方式是给函数传入所有参数，包括类型参数：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}
// ---cut---
let output = identity<string>("myString");
//       ^?
```

这里我们显式的给`Type`设置成`string`作为函数调用的第一个采纳数，注意在参数周围使用`<>`而不是`()`来表示。

第二个方式也许是最常见的。这里我们使用 _类型参数断言_ — 意思是，我们想要编辑器基于我们传递的参数自动设置`Type`的值“

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}
// ---cut---
let output = identity("myString");
//       ^?
```

注意我们不必显式的在尖括号(`<>`)中显式的传递类型；编译器只是查看了值`myString`，并且将`Type`设置为它的类型，
虽然类型参数推断可以成为保持代码更短和更具可读性的有用工具，但是当编译器无法推断类型时，您看您需要像我们在上一个示例中所做的那样显式传递类型参数，这看您发生在更复杂的示例中。

## 使用泛型类型变量

当我们开始使用泛型，你将会注意到当你创建像`identity`的函数时候，编译器将会强迫你正确使用函数体重的任何通用类型参数。（但是你不知道这些泛型类型）
也就是说，您实际上将这些参数视为可以使任何和所有类型。


让我们看看之前的`identity`函数：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}
```

如果我们还想在每次调用里打印`agr`参数的长度呢？
我们可能会想这样写：

```ts twoslash
// @errors: 2339
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
  return arg;
}
```

当我们这样做，编译器将会给我们报错我们正在使用`arg`的`.length`属性，但是没有地方指明`agr`有这个属性。
记住，我们前面说这些类型变量代表任何和所有类型，所有有人使用这个函数可能传入一个`number`参数，他没有`.length`属性。

让我们假设我们正在想要参数是`Type`数字而不是`Type`。因为我们使用数组，`.length`属性应该是有效的。
我们可以像创建其他类型的数组一样来描述它：

```ts twoslash {1}
function loggingIdentity<Type>(arg: Type[]): Type[] {
  console.log(arg.length);
  return arg;
}
```

你可以将`loggingIdentity`的类型理解为“这个泛型函数`loggingIdentity`接收一个类型参数`Type`，并且一个入参`arg`是一个`Type`的数组类型，并且返回`Type`的数组类型”
如果我们传入一个数字数组，我们会一个数字数组，因为`Type`会被绑定为`number`。
这个允许我们将泛型类型变量`Type`作为我们正在使用的类型的一部分，而不是整个类型，从而为我们提供更大的灵活性。

我们可以可选的像下面写例子：

```ts twoslash {1}
function loggingIdentity<Type>(arg: Array<Type>): Array<Type> {
  console.log(arg.length); // Array has a .length, so no more error
  return arg;
}
```

你可能已经从其他语言熟悉了这种类型风格。
在下一节，我们将会介绍如何创建自己的泛型类型，例如`Array<Type>`
You may already be familiar with this style of type from other languages.
In the next section, we'll cover how you can create your own generic types like `Array<Type>`.

## 泛型类型

在上一节，我们创建了泛型恒等函数，能够在广泛的类型上使用。
这一节，我们将会探索函数本身的类型，并且如何去创造泛型接口。

泛型函数的类型（就是下面`<Type>(arg: Type) => Type`这个东西）就像哪些非泛型函数一样，先列出参数，类似普通函数声明：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: <Type>(arg: Type) => Type = identity;
```

我们也可以为类型中的泛型类型参数参数使用不同的名字，只要类型变量的数量和使用方法是一致的就行。

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: <Input>(arg: Input) => Input = identity;
```

我们也可以通过对象文本类型的调用签名的形式写一个泛型类型：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: { <Type>(arg: Type): Type } = identity;
```

这个可以引导我们到第一个泛型接口。
让我们从前面的例子中获取对象文本值然后移动到接口上：

```ts twoslash
interface GenericIdentityFn {
  <Type>(arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

在类似的示例中，我们可能希望将泛型参数移动为整个接口的参数。
这让我们可以看到我们泛型的类型（例如`Dictionary<string>`而不仅仅是`Dictionary`）。
这个让类型参数对于其他接口的所有成员是可见的。（就是外面直接指明类型参数，就知道接口里用到这些参数的类型了）

```ts twoslash
interface GenericIdentityFn<Type> {
  (arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

注意我们的例子已经变得稍微不一样了。
我们现在有一个**非泛型**函数签名，他是泛型类型的一部分，却不是描述泛型函数的。
当我们使用`GenericIdentityFn`的时候，我们现在也需要相关的类型参数（这里：`number`），有效的锁定了底层调用签名将使用的内容。
理解何时将类型参数直接使用在调用签名上和何时将他将其放在接口本身将有助于描述类型的哪些方面是通用的（就是上面两种形式的用法的描述角度不同）。

除了泛型接口，我们还可以创建泛型类。
请注意，无法创建泛型枚举和泛型命名空间。

## 泛型类

一个泛型类和泛型接口有相似的形状。
泛型类有泛型类型参数列表在尖括号（`<>`）里，在类的名字后面。

```ts twoslash
// @strict: false
class GenericNumber<NumType> {
  zeroValue: NumType;
  add: (x: NumType, y: NumType) => NumType;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function (x, y) {
  return x + y;
};
```

这个是`GenericNumber`类的直接用法，但是你可能已经注意到，只能限制到使用`number`类型。
我们本可以使用`string`甚至更复杂的对象。

```ts twoslash
// @strict: false
class GenericNumber<NumType> {
  zeroValue: NumType;
  add: (x: NumType, y: NumType) => NumType;
}
// ---cut---
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function (x, y) {
  return x + y;
};

console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

就像接口一样，在类本身后面添加类型参数让我们确保类里面的属性是用相同的类型工作。

正如我们在[类的章节里](/docs/handbook/2/classes.html)说到的，一个类对它的类型有两面性：静态类型和实例类型。
泛型类型只能覆盖实例类型而不能覆盖静态类型，所以当使用class的时候，静态成员不能使用class的类型参数。

## 泛型限制

如果你在还记得更早的例子，你可能像有写一个泛型函数来说，该函数使用一组类型，然后你可能获取这个类型能力的 _某些_ 知识。 
在我们的`loggingIdentity`例子里，我们想要获取`arg`的`.length`属性，但是编译器不能保证每种类型都有`.length`属性，所以它浸膏我们不能做这种假设。

```ts twoslash
// @errors: 2339
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
  return arg;
}
```

与其要能和所有类型工作，我们想要约束函数和任意有`.length`属性的类型工作。
只要类型有成员，我们允许它，但它至少需要有这个成员。
为了实现这个，我们必须给`Type`类型列出需求的约束。

为了实现这个，我们创建一个描述我们约束的接口。
这里，我们会创建一个接口有一个`.length`属性，然后我们会用这接口和`extends`关键字表示我们的约束：

```ts twoslash
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length); // Now we know it has a .length property, so no more error
  return arg;
}
```

因为泛型函数现在被约束了，它不再能和其他任意类型工作了：

```ts twoslash
// @errors: 2345
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length);
  return arg;
}
// ---cut---
loggingIdentity(3);
```

相反，我们需要传入一些值，拥有必须的属性：

```ts twoslash
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length);
  return arg;
}
// ---cut---
loggingIdentity({ length: 10, value: 3 });
```

## 在泛型约束里使用类型

你可以声明一个类型参数来约束其他类型参数。
例如，这里我们想要给从一个给定名称的对象中获取一个属性。
我们想要确保我们不会意外抓取`obj`上不存在的属性，因此我们将在这两种类型之间放置一个约束（下面用了`keyof`）：

```ts twoslash
// @errors: 2345
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a");
getProperty(x, "m");
```

## 在泛型里用类类型

当在TypeScript里使用泛型创建工厂时，需要其构造函数引用类类型。例如（更复杂的用法），

```ts twoslash
function create<Type>(c: { new (): Type }): Type {
  return new c();
}
```

一个更高阶的例子是使用原型属性推断和约束构造函数和类实例类型之间的关系。

```ts twoslash
// @strict: false
class BeeKeeper {
  hasMask: boolean = true;
}

class ZooKeeper {
  nametag: string = "Mikle";
}

class Animal {
  numLegs: number = 4;
}

class Bee extends Animal {
  keeper: BeeKeeper = new BeeKeeper();
}

class Lion extends Animal {
  keeper: ZooKeeper = new ZooKeeper();
}

function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}

createInstance(Lion).keeper.nametag;
createInstance(Bee).keeper.hasMask;
```

此模式用于支持[mixins](/docs/handbook/mixins.html)设计模式。
