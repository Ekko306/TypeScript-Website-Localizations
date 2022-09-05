---
title: Classes
layout: docs
permalink: /zh/docs/handbook/2/classes.html
oneline: "How classes work in TypeScript"
---

<blockquote class='bg-reading'>
  <p>背景知识阅读：<br /><a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes'>Classes (MDN)</a></p>
</blockquote>

TypeScript对于ES2015里的`class`关键字完全支持。

正如其他JavaScript其他语言特性，TypeScript也增加了类型注释和其他的语法允许你让类和其他类型表示联系。

## 类成员

这里是最基础的类 — 一个空类。

```ts twoslash
class Point {}
```

这个类还没有太大用，所以让我们添加一些成员。

### 字段

一个字段声明会在类上创建一个公共可写的属性：

```ts twoslash
// @strictPropertyInitialization: false
class Point {
  x: number;
  y: number;
}

const pt = new Point();
pt.x = 0;
pt.y = 0;
```

和其他位置一样，上面的类型注释（x: number的 number ）是可选的，但是如果未指定，它将是隐式的`any`类型。

字段也可以有 _初始值_ ；这些值将在类被实例化的时候自动执行：

```ts twoslash
class Point {
  x = 0;
  y = 0;
}

const pt = new Point();
// Prints 0, 0
console.log(`${pt.x}, ${pt.y}`);
```

就像`const`，`let`和`var`，一个类属性的初始值将会被用来推断他的类型：

```ts twoslash
// @errors: 2322
class Point {
  x = 0;
  y = 0;
}
// ---cut---
const pt = new Point();
pt.x = "0";
```

#### `--strictPropertyInitialization`

[`strictPropertyInitialization`](/tsconfig#strictPropertyInitialization)设置控制了在`constructor`里是否需要初始化类的字段。

```ts twoslash
// @errors: 2564
class BadGreeter {
  name: string;
}
```

```ts twoslash
class GoodGreeter {
  name: string;

  constructor() {
    this.name = "hello";
  }
}
```

注意字段需要 _在构造函数里_ 自己初始化。
TypeScript**不会分析**您从构造函数里调用的方法来检查初始化，因为派生类可能会覆盖这些方法并且无法初始化成员。（没给例子，但可以注意，就是派生是合法的，可能导致有些初始化出问题。）

如果你打算通过构造函数以外的方式明确地初始化一个字段（例如，可能一个外部库正在为你填充你的类的一部分），你可以使用 _确定赋值断言运算符_，`!`：
```ts twoslash
class OKGreeter {
  // Not initialized, but no error
  name!: string;
}
```

### `readonly`

字段可以用`readonly`修饰器添加前缀。
这可以防止在构造函数之外的时候对字段进行赋值。

```ts twoslash
// @errors: 2540 2540
class Greeter {
  readonly name: string = "world";

  constructor(otherName?: string) {
    if (otherName !== undefined) {
      this.name = otherName;
    }
  }

  err() {
    this.name = "not ok";
  }
}
const g = new Greeter();
g.name = "also not ok";
```

### 构造函数

<blockquote class='bg-reading'>
   <p>背景知识阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor'>Constructor (MDN)</a><br/>
   </p>
</blockquote>

类的构造函数和函数非常相似。
你可以使用类型注释，默认值和重载来增加参数：

```ts twoslash
class Point {
  x: number;
  y: number;

  // Normal signature with defaults
  constructor(x = 0, y = 0) {
    this.x = x;
    this.y = y;
  }
}
```

```ts twoslash
class Point {
  // Overloads
  constructor(x: number, y: string);
  constructor(s: string);
  constructor(xs: any, y?: any) {
    // TBD
  }
}
```

类构造签名和函数构造签名有一些不同：

- 构造函数不能有类型参数（就是尖括号<>） — 因为这些属于外部的类型定义，我们之后会讲到。
- 构造函数不能返回类型声明 — 因为类的实例就是返回的类型。

#### Super关键字调用

正如JavaScript一样，如果你有一个基类，你需要在你的构造函数里调用`super();`，然后才能使用对应的`this.`成员：

```ts twoslash
// @errors: 17009
class Base {
  k = 4;
}

class Derived extends Base {
  constructor() {
    // Prints a wrong value in ES5; throws exception in ES6
    console.log(this.k);
    super();
  }
}
```

忘记调用`super`是在JavaScript很容易犯的错误，但是TypeScript会告诉你有些是必须的。

### 方法

<blockquote class='bg-reading'>
   <p>背景知识阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions'>Method definitions</a><br/>
   </p>
</blockquote>
一个类的函数属性叫做 _方法_。
方法可以使用所有与函数和构造函数相同的类型注释：
A function property on a class is called a _method_.

```ts twoslash
class Point {
  x = 10;
  y = 10;

  scale(n: number): void {
    this.x *= n;
    this.y *= n;
  }
}
```

除了标准的类型注释，TypeScript没有为方法添加任何新的东西。

注意在方法内部，通过`this.`来获取字段和其他方法仍然是必须的。
一个在方法体里不合格的参数名可能会指向作用域里的其他东西（不用this，直接操作变量可能是外部作用域的）：

```ts twoslash
// @errors: 2322
let x: number = 0;

class C {
  x: string = "hello";

  m() {
    // This is trying to modify 'x' from line 1, not the class property
    x = "world";
  }
}
```

### Getters / Setters

类也可以有 _属性访问器_ ：

```ts twoslash
class C {
  _length = 0;
  get length() {
    return this._length;
  }
  set length(value) {
    this._length = value;
  }
}
```

> 注意没有额外逻辑的get/set对于字段很少有用。
> 如果您不需要在get/set操作期间有额外的逻辑，则可以暴露公共的字段。

TypeScript对于属性访问器有特殊的推断规则（下面的原则是引出下面ts里的属性访问器的）：

- 如果`get`存在，但是不存在`set`，这个属性自动变成`readonly`的
- 如果setter参数没有被指定，他从getter的返回值推断。
- getters和setter必须要有相同的 [成员可视性](#member-visibility)

从[TypeScript 4.3](https://devblogs.microsoft.com/typescript/announcing-typescript-4-3/)开始，对getting和setting使用不同类型的属性访问器已经支持了。

```ts twoslash
class Thing {
  _size = 0;

  get size(): number {
    return this._size;
  }

  set size(value: string | number | boolean) {
    let num = Number(value);

    // Don't allow NaN, Infinity, etc

    if (!Number.isFinite(num)) {
      this._size = 0;
      return;
    }

    this._size = num;
  }
}
```

### 索引签名

类可以声明索引签名，这些和 [对象的索引签名](/docs/handbook/2/objects.html#index-signatures)工作方式类似：

```ts twoslash
class MyClass {
  [s: string]: boolean | ((s: string) => boolean);

  check(s: string) {
    return this[s] as boolean;
  }
}
```

因为索引签名类型需要获取方法的类型，使用这些索引类型那么的时候不是很容易。
通常最好将索引数据存在其他地方而不是类实例本身。

## 类继承

就像其他用于面向对象特性的语言，JavaScript里的类可以从基础类继承。

### `implements`语法

你可以使用一个`implements`语法来检查一个类满足特定的`interface`。
如果一个类未能正确实现它，则会发出错误：

```ts twoslash
// @errors: 2420
interface Pingable {
  ping(): void;
}

class Sonar implements Pingable {
  ping() {
    console.log("ping!");
  }
}

class Ball implements Pingable {
  pong() {
    console.log("pong!");
  }
}
```

类可能也会实现多个接口，例如`class C implements A, B{`。

#### 注意

很重要的一点是：一个`implements`语法只会检查这个类可以被当做接口类型。
它 _完全_ 不会改变类或者他的成员。
一个常见的错误来源是假设`implements`子句会改变类类型——它不会！

```ts twoslash
// @errors: 7006
interface Checkable {
  check(name: string): boolean;
}

class NameChecker implements Checkable {
  check(s) {
    // Notice no error here
    return s.toLowercse() === "ok";
    //         ^?
  }
}
```

在上面例子里，我们可能想要`s`的类型会被`check`的参数`name: string`影响。
但他不会——`implements`子句不会改变检查类主体或推断类型的方式。

同样的，使用可选属性实现接口不会创建改属性（interface只会检查是否满足）：

```ts twoslash
// @errors: 2339
interface A {
  x: number;
  y?: number;
}
class C implements A {
  x = 0;
}
const c = new C();
c.y = 10;
```

### `extends`语法

<blockquote class='bg-reading'>
   <p>背景阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends'>extends keyword (MDN)</a><br/>
   </p>
</blockquote>

类可能从一个基类`extend`出来。
一个 派生类有它基类的所有属性和方法吗，还可以定义其他额外成员。

```ts twoslash
class Animal {
  move() {
    console.log("Moving along!");
  }
}

class Dog extends Animal {
  woof(times: number) {
    for (let i = 0; i < times; i++) {
      console.log("woof!");
    }
  }
}

const d = new Dog();
// Base class method
d.move();
// Derived class method
d.woof(3);
```

#### 覆盖方法

<blockquote class='bg-reading'>
   <p>背景知识阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super'>super keyword (MDN)</a><br/>
   </p>
</blockquote>

一个派生类也可以重写基类的字段或属性。
你可以使用`super.`的语法获取基类的方法。
注意因为JavaScript的类是一个简单的查找对象（语法糖），所以没有“超级字段”的概念。

TypeScript强迫一个派生类始终是基类的子类型。

例如，这里有一个合法的方式重写一个方法：

```ts twoslash
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) {
      super.greet();
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}

const d = new Derived();
d.greet();
d.greet("reader");
```

一个派生类遵循基类的约束很重要。
记住通过基类来引用来推断派生类实例是很常见的（而且总是合法的！）：

```ts twoslash
class Base {
  greet() {
    console.log("Hello, world!");
  }
}
declare const d: Base;
// ---cut---
// Alias the derived instance through a base class reference
const b: Base = d;
// No problem
b.greet();
```

如果`Dervied`派生类不遵循`Base`基类约束呢？（下面例子是因为ts保证你派生类要满足基类约束否则报错，所以用起来就可以使用用基类推断派生类（上面用法））

```ts twoslash
// @errors: 2416
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  // Make this parameter required
  greet(name: string) {
    console.log(`Hello, ${name.toUpperCase()}`);
  }
}
```

如果我们忽略错误编ts代码，然后这个例子会崩溃：

```ts twoslash
declare class Base {
  greet(): void;
}
declare class Derived extends Base {}
// ---cut---
const b: Base = new Derived();
// Crashes because "name" will be undefined
b.greet();
```

#### 仅类型字段声明

当`target >= ES2022`或者 [`useDefineForClassFields`](/tsconfig#useDefineForClassFields) 配置项为`true`时，类的字段会在父类构造函数完成后再初始化，就表示自己的类字段会覆盖父类的字段（就是ts假设你派生类再定义resident会覆盖父类的，你不想让ts误解，就说明delcare，表明还是用父类的）。当你只想在派生字段之后重新定义一个更正确的字段会报错。为了解决这个情况，你可以写`declare`来告诉TypeScript这个字段声明不会对运行时有影响。

```ts twoslash
interface Animal {
  dateOfBirth: any;
}

interface Dog extends Animal {
  breed: any;
}

class AnimalHouse {
  resident: Animal;
  constructor(animal: Animal) {
    this.resident = animal;
  }
}

class DogHouse extends AnimalHouse {
  // Does not emit JavaScript code,
  // only ensures the types are correct
  declare resident: Dog;
  constructor(dog: Dog) {
    super(dog);
  }
}
```

#### 初始化顺序

JavaScript里类的初始化在某些情况出乎意料。
我们可以看下面例子：

```ts twoslash
class Base {
  name = "base";
  constructor() {
    console.log("My name is " + this.name);
  }
}

class Derived extends Base {
  name = "derived";
}

// Prints "base", not "derived"
const d = new Derived();
```

这里发生了什么？

按照JavaScript的定义方式，类初始化的顺序是：

- 初始化基类的字段
- 基类的构造函数执行
- 初始化派生类的字段
- 派生类的构造函数执行

这个意味着基类构造函数在自己的构造函数里看见了自己的`name`值，因为派生类字段初始化还没有运行。

#### 继承内置类型

> 提示：如果你不打算从例如`Array`，`Error`和`Map`等内置类型继承，或者你的编译目标是`ES6`/`ES2015`或者更高，你可以跳过这一节。

在ES2015里，返回对象的构造函数会隐式地将`this`值替换给`super(...)`的任何调用者。
生成的构造函数代码必须捕获`super(...)`的任何潜在返回值并将其替换为`this`。

因此，子类的`Error`，`Array`和其他类似的不能按照预期工作。
这是因为`Error`，`Array`等的构造函数使用了ECMAScript 6的`new.target`来调整原型链；
但是在ECMAScript 5中，无法确保`new.target`的值。
默认情况下，其他更低级编译器通常具有相同的限制。

比如像下面的子类：

```ts twoslash
class MsgError extends Error {
  constructor(m: string) {
    super(m);
  }
  sayHello() {
    return "hello " + this.message;
  }
}
```

你可能会发现（在ES3啥的ts的target）：

- 子类调用构造函数后返回的对象上的方法可能是`undefined`的，所以调用`sayHello`将会报错。
- `instanceof`将会在子类的实例和实例之间中断，因此`(new MsgError()) instanceof MsgError`会返回`false`。

作为建议，可以在人后`super()`调用后立即手动调整原型。（适配低版本浏览器）

```ts twoslash
class MsgError extends Error {
  constructor(m: string) {
    super(m);

    // Set the prototype explicitly.
    Object.setPrototypeOf(this, MsgError.prototype);
  }

  sayHello() {
    return "hello " + this.message;
  }
}
```

然而`MsgError`的任何子类也要手动调整原型。
对于哪些不支持 [`Object.setPrototypeOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf)的运行时，你可能要使用[`__proto__`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)。

不幸的是，[这些变通方法不适用与Internet Explorer 10和更早版本](<https://msdn.microsoft.com/en-us/library/s4esdbwz(v=vs.94).aspx>)。
我们可以手动从原型方法本身复制到实例本身（即`MsgError.prototype`到`this`），但是原型链本身不能修复。（就是说子类也要手动调整原型）

## 成员可视性

你可以使用TypeScript来控制是否一个方法或者属性对于类外的代码是否是可见的。

### `public`

类成员的默认可视性是`public`。
一个`public`成员可以被任何位置获取：

```ts twoslash
class Greeter {
  public greet() {
    console.log("hi!");
  }
}
const g = new Greeter();
g.greet();
```

因为`public`已经是默认的可视性修改器，你 _不需要_ 在类成员前面写，但是可能对于风格和可读性可以加上。

### `protected`

`protected`成员只是对他们声明它们的类的子类可见。 

```ts twoslash
// @errors: 2445
class Greeter {
  public greet() {
    console.log("Hello, " + this.getName());
  }
  protected getName() {
    return "hi";
  }
}

class SpecialGreeter extends Greeter {
  public howdy() {
    // OK to access protected member here
    console.log("Howdy, " + this.getName());
    //                          ^^^^^^^^^^^^^^
  }
}
const g = new SpecialGreeter();
g.greet(); // OK
g.getName();
```

#### 暴露`protected`成员

派生类需要遵循其基类的约束，但是可以选择公开更多功能的基类子类型。
这包括将`protected`成员设置为`public`：

```ts twoslash
class Base {
  protected m = 10;
}
class Derived extends Base {
  // No modifier, so default is 'public'
  m = 15;
}
const d = new Derived();
console.log(d.m); // OK
```

注意`Derived`类已经能够被随意的读写`m`，所以这不是显式的修改这种情况的“安全性”。
要注意的是在派生类里，如果这种暴露不是故意的，我们需要小心重复`protected`修饰符。（？理解有问题）

#### 跨实例的`protected`访问

不同的面向对象语言对于是否能够访问基类引用的`protected`成员有不同的合法性：

```ts twoslash
// @errors: 2446
class Base {
  protected x: number = 1;
}
class Derived1 extends Base {
  protected x: number = 5;
}
class Derived2 extends Base {
  f1(other: Derived2) {
    other.x = 10;
  }
  f2(other: Base) {
    other.x = 10;
  }
}
```

例如Java认为这是合法的。
然而，C#和C++认为这种代码是非法的。

（感觉下面的话有问题，和代码对应不上，或者理解很奇怪，可能和上面代码无关，单纯讨论其他理论）
TypeScript选择C#和C++做法，因为在`Derived2`中访问`x`应该只在`Derived2`的子类是合法的，`Dervied1`不是`Derived2`的子类，此时`Dervied2`不能获取`Dervied1`的`x`。
此外，有种情况，如果通过`Derived1`引用不能访问到`Dervied2`的`x`，name通过`Dervied1`的基类也是不能访问到的。（可能套路各种层级情况）

另请参阅 [为什么我不能从派生类访问受保护的成员？](https://blogs.msdn.microsoft.com/ericlippert/2005/11/09/why-cant-i-access-a-protected -member-from-a-derived-class/) 这解释了更多 C# 的推理。

### `private`

`private`和`protected`很像，但是连子类获取成员也不允许：

```ts twoslash
// @errors: 2341
class Base {
  private x = 0;
}
const b = new Base();
// Can't access from outside the class
console.log(b.x);
```

```ts twoslash
// @errors: 2341
class Base {
  private x = 0;
}
// ---cut---
class Derived extends Base {
  showX() {
    // Can't access in subclasses
    console.log(this.x);
  }
}
```

因为`private`成员不能在派生类（子类）里访问，一个派生类不能提升他的可视性（像前面提升protected可视性是可以的）：

```ts twoslash
// @errors: 2415
class Base {
  private x = 0;
}
class Derived extends Base {
  x = 1;
}
```

#### 跨实例的`private`访问

（？这个和上面protect一样，有点迷，重点理解★）
不同的OOP语言对于同一类的不同实例是否可以访问彼此的`private`成员存在分歧。
虽然Java、C#、C++、Swift和PHP等语言允许这样做，但Ruby不允许。

TypeScript确实允许跨实例`private`访问：

```ts twoslash
class A {
  private x = 10;

  public sameAs(other: A) {
    // No error
    return other.x === this.x;
  }
}
```

#### 注意事项

就像TypeScript类型系统的其他方面一样，`private`和`protected`[仅在类型检查期间强制执行](https://www.typescriptlang.org/play?removeComments=true&target=99&ts=4.3.4#code/PTAEGMBsEMGddAEQPYHNQBMCmVoCcsEAHPASwDdoAXLUAM1K0gwQFdZSA7dAKWkoDK4MkSoByBAGJQJLAwAeAWABQIUH0HDSoiTLKUaoUggAW+DHorUsAOlABJcQlhUy4KpACeoLJzrI8cCwMGxU1ABVPIiwhESpMZEJQTmR4lxFQaQxWMm4IZABbIlIYKlJkTlDlXHgkNFAAbxVQTIAjfABrAEEC5FZOeIBeUAAGAG5mmSw8WAroSFIqb2GAIjMiIk8VieVJ8Ar01ncAgAoASkaAXxVr3dUwGoQAYWpMHBgCYn1rekZmNg4eUi0Vi2icoBWJCsNBWoA6WE8AHcAiEwmBgTEtDovtDaMZQLM6PEoQZbA5wSk0q5SO4vD4-AEghZoJwLGYEIRwNBoqAzFRwCZCFUIlFMXECdSiAhId8YZgclx0PsiiVqOVOAAaUAFLAsxWgKiC35MFigfC0FKgSAVVDTSyk+W5dB4fplHVVR6gF7xJrKFotEk-HXIRE9PoDUDDcaTAPTWaceaLZYQlmoPBbHYx-KcQ7HPDnK43FQqfY5+IMDDISPJLCIuqoc47UsuUCofAME3Vzi1r3URvF5QV5A2STtPDdXqunZDgDaYlHnTDrrEAF0dm28B3mDZg6HJwN1+2-hg57ulwNV2NQGoZbjYfNrYiENBwEFaojFiZQK08C-4fFKTVCozWfTgfFgLkeT5AUqiAA)。（这里例子说了js和ts里的private区别，js是强制的，ts却是报错的，如果忽略错误还是让你编译过去，但是private啥的可能表象编译过去就没了）

这个意味着JavaScript运行时里像`in`或简单的属性查找仍然可以访问`private`或`protected`成员（忽略错误变异出去了就没有private啥约束了）：

```ts twoslash
class MySafe {
  private secretKey = 12345;
}
```

```js
// In a JavaScript file...
const s = new MySafe();
// Will print 12345
console.log(s.secretKey);
```

`private`还允许在类型检查期间使用括号表示法进行访问（`console.log(s["secretKey"]);`这种）。`private`声明的字段可能更容易访问，例如单元测试，缺点是这些字段是 _软私有的_，并不严格执行隐私。

```ts twoslash
// @errors: 2341
class MySafe {
  private secretKey = 12345;
}

const s = new MySafe();

// Not allowed during type checking
console.log(s.secretKey);

// OK
console.log(s["secretKey"]);
```

不像TypeScript的`private`，JavaScript的[私有字段](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields) (`#`)在编译之后会保持私有并且不提供前面提到的转义舱口（括号字符串访问），使他们成为 _硬私有的_。

```ts twoslash
class Dog {
  #barkAmount = 0;
  personality = "happy";

  constructor() {}
}
```

```ts twoslash
// @target: esnext
// @showEmit
class Dog {
  #barkAmount = 0;
  personality = "happy";

  constructor() {}
}
```

当编译目标是ES2021的更低版本，TypeScript会把`#`的位置编译成弱映射。（就是老版本是下面这种方式，不是js里的#强制约束，但是选择最最新版本比如esnext，会变成上面强私有约束）
When compiling to ES2021 or less, TypeScript will use WeakMaps in place of `#`.

```ts twoslash
// @target: es2015
// @showEmit
class Dog {
  #barkAmount = 0;
  personality = "happy";

  constructor() {}
}
```

如果你想要保护类中的值面授恶意行为者的侵害，你应该使用提供应运行时隐私机制，例如闭包，WeakMap或私有字段。要注意这些运行时添加的隐私检查可能影响性能（大量数据）。

## 静态成员

<blockquote class='bg-reading'>
   <p>背景知识阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static'>Static Members (MDN)</a><br/>
   </p>
</blockquote>

类可能有`static`成员。
这些成员不与类的特定实例相关联。
他们可以通过类构造函数对象本身访问：

```ts twoslash
class MyClass {
  static x = 0;
  static printX() {
    console.log(MyClass.x);
  }
}
console.log(MyClass.x);
MyClass.printX();
```

静态成员也可以使用相同的`public`，`protected`和`private`可视性修饰器：

```ts twoslash
// @errors: 2341
class MyClass {
  private static x = 0;
}
console.log(MyClass.x);
```

静态成员也是可以被继承的：

```ts twoslash
class Base {
  static getGreeting() {
    return "Hello world";
  }
}
class Derived extends Base {
  myGreeting = Derived.getGreeting();
}
```

### 特殊静态名称

从`Function`原型覆盖属性通常是不安全/不允许的。
因为类本身就是可以用`new`调用的“函数”，所以不能使用一些**特定**的`static`名称。
像`name`、`length`和`call`这样的属性不能定义为`static`成员：

```ts twoslash
// @errors: 2699
class S {
  static name = "S!";
}
```

### 为什么没有静态类（很少）？

（前面是静态成员）
TypeScript（和JavaScript）没有和C#等拥有的的`static class`静态类构造形式。

哪些构造方式存在的原因 _只是_ 因为哪些语言强迫所有数据和函数都在类里实现；因为TypeScript没有这种限制，所以不需要静态类。
在JavaScript/TypeScript里，一个只拥有一个实例的类会被当做一个普通的 _对象_ 。

例如，我们不需要在TypeScript里使用“静态类”语法因为一个普通对象（甚至高阶函数）也可以完成同样的工作：

```ts twoslash
// Unnecessary "static" class
class MyStaticClass {
  static doSomething() {}
}

// Preferred (alternative 1)
function doSomething() {}

// Preferred (alternative 2)
const MyHelperObject = {
  dosomething() {},
};
```

## 类中的`static`块

静态块允许你编写具有自己范围的语句序列，这些语句中可以访问包含类中的私有字段。这意味着我们可以编写具有语句的所有功能的初始化代码，不会泄露变量，并且可以完全访问我们类的内部结构。

```ts twoslash
declare function loadLastInstances(): any[]
// ---cut---
class Foo {
    static #count = 0;

    get count() {
        return Foo.#count;
    }

    static {
        try {
            const lastInstances = loadLastInstances();
            Foo.#count += lastInstances.length;
        }
        catch {}
    }
}
```

## 泛型类

类，很像接口，也可以有泛型。
当一个泛型类被`new`实例化的时候，他的类型参数会和函数调用一样被推断出来：

```ts twoslash
class Box<Type> {
  contents: Type;
  constructor(value: Type) {
    this.contents = value;
  }
}

const b = new Box("hello!");
//    ^?
```

类可以像接口一样使用泛型约束和默认值（泛型默认值？）。

### 静态成员的类型参数

下面代码不合法，原因可能并不明显：

```ts twoslash
// @errors: 2302
class Box<Type> {
  static defaultValue: Type;
}
```

请记住，类型总是被擦除的！
在运行时，只会有 _一个_ `Box.defaultValue`属性槽。
这意味着设置 `Box<string>.defaultValue`（如果可以），将 _也_ 会改变 `Box<number>.defaultValue` — 这点不好。
泛型类的`static`成员永远不能引用类的类型参数。

## 类中运行时的`this`

<blockquote class='bg-reading'>
   <p>背景知识阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this'>this keyword (MDN)</a><br/>
   </p>
</blockquote>

记住TypeScript不会改变JavaScript运行时行为是很重要的，并且JavaScript以具有一些特殊的运行时行为而闻名。

JavaScript对`this`的处理确实不寻常：

```ts twoslash
class MyClass {
  name = "MyClass";
  getName() {
    return this.name;
  }
}
const c = new MyClass();
const obj = {
  name: "obj",
  getName: c.getName,
};

// Prints "obj", not "MyClass"
console.log(obj.getName());
```

长话短说，默认情况下，函数内的`this`的值取决于 _如何调用函数_ 。
在这个例子里，因为函数通过`obj`引用调用，他的`this`值是`obj`而不是类实例的`this`值。

这很少情况是你想要发生的！
TypeScript 提供了一些减轻或防止此类错误的方法。

### 箭头函数

<blockquote class='bg-reading'>
   <p>背景知识阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions'>Arrow functions (MDN)</a><br/>
   </p>
</blockquote>

如果你有一个经常以丢失其`this`上下文的方式调用的函数，则使用箭头函数属性而不是方法定义的方式是有意义的。（方法定义的方式：`getName() {return this.name;}`）

```ts twoslash
class MyClass {
  name = "MyClass";
  getName = () => {
    return this.name;
  };
}
const c = new MyClass();
const g = c.getName;
// Prints "MyClass" instead of crashing
console.log(g());
```

这里有一些权衡：

- `this`的值被保证在运行时有正确的值，即使对没有被TypeScript检查的代码
- 这个会使用更多内存，因为每个函数实例将会有这种方式定义每个函数的拷贝。
- 你不能在派生类里使用`super.getName`，因为原型链中没有条目可以从中获取基类方法。

### `this`参数

在方法或函数定义里，TypeScript里一个叫做`this`的初始参数有特殊意义。
这个参数会在编译过程中被删除：

```ts twoslash
type SomeType = any;
// ---cut---
// TypeScript input with 'this' parameter
function fn(this: SomeType, x: number) {
  /* ... */
}
```

```js
// JavaScript output
function fn(x) {
  /* ... */
}
```

TypeScript会检查调用带有`this`参数的函数是否在正确的上下文中完成。
因为TypeScript没有选择箭头函数（考虑副作用），而是显式的在方法定义中添加一个`this`参数，以静态强制方法被正确调用：

```ts twoslash
// @errors: 2684
class MyClass {
  name = "MyClass";
  getName(this: MyClass) {
    return this.name;
  }
}
const c = new MyClass();
// OK
c.getName();

// Error, would crash
const g = c.getName;
console.log(g());
```

此方法与箭头函数方法进行了相反的权衡：

- JavaScript调用者可能仍然在没有意识的情况下使用类方法（像上面的错误使用，ts显式的报错，但是箭头函数默认修正了）
- 每个类定义只分配了一个函数，而不是每个实例一个
- 基本方法定义仍然可以通过`super`调用。

## `this` 类型

（this可以作为参数也可以作为一个特殊的类型）
在类里，一种被称为`this`的特殊类型 _动态地_ 引用当前类的类型。让我们看看这有什么用处：

<!-- prettier-ignore -->
```ts twoslash
class Box {
  contents: string = "";
  set(value: string) {
//  ^?
    this.contents = value;
    return this;
  }
}
```

这里TypeScript推断`set`的返回值是`this`，而不是`Box`
现在让我们构建一个`Box`的子类：（发现set返回值指向了`ClearbleBox`子类，比较正确）

```ts twoslash
class Box {
  contents: string = "";
  set(value: string) {
    this.contents = value;
    return this;
  }
}
// ---cut---
class ClearableBox extends Box {
  clear() {
    this.contents = "";
  }
}

const a = new ClearableBox();
const b = a.set("hello");
//    ^?
```

你也可以再类型注释中使用`this`：

```ts twoslash
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}
```

这个和写`other: Box`不同——如果你有个派生类型，他的`sameAs`方法将只接收有相同派生类型的实例。（就是`DerivedBox`类里的，sameAs的参数变成了`other: DerivedBox`）：

```ts twoslash
// @errors: 2345
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}

class DerivedBox extends Box {
  otherContent: string = "?";
}

const base = new Box();
const derived = new DerivedBox();
derived.sameAs(base);
```

### `this` - 基本的类型保护

你可以在类和接口的方法返回值位置使用使用`this is Type`。当和类型缩小结合时（例如`if`声明语句），目标对象的类型将会缩小为对应的`Type`。

<!-- prettier-ignore -->
```ts twoslash
// @strictPropertyInitialization: false
class FileSystemObject {
  isFile(): this is FileRep {
    return this instanceof FileRep;
  }
  isDirectory(): this is Directory {
    return this instanceof Directory;
  }
  isNetworked(): this is Networked & this {
    return this.networked;
  }
  constructor(public path: string, private networked: boolean) {}
}

class FileRep extends FileSystemObject {
  constructor(path: string, public content: string) {
    super(path, false);
  }
}

class Directory extends FileSystemObject {
  children: FileSystemObject[];
}

interface Networked {
  host: string;
}

const fso: FileSystemObject = new FileRep("foo/bar.txt", "foo");

if (fso.isFile()) {
  fso.content;
// ^?
} else if (fso.isDirectory()) {
  fso.children;
// ^?
} else if (fso.isNetworked()) {
  fso.host;
// ^?
}
```

基于this的类型保护的一个常见用例是允许对特定字段进行延迟验证。例如下面例子，当`hasValue`被验证为true时，会从box实例里移除`undefined`的类型：

```ts twoslash
class Box<T> {
  value?: T;

  hasValue(): this is { value: T } {
    return this.value !== undefined;
  }
}

const box = new Box();
box.value = "Gameboy";

box.value;
//  ^?

if (box.hasValue()) {
  box.value;
  //  ^?
}
```

## 参数属性

TypeScript提供特殊的语法，将构造函数参数转变为具有相同名称和值的类属性。
这个方式叫做 _参数属性_ 并且通过在构造函数参数前加上可见性修饰符 `public`，`private`，`protected`或`readonly`之一来创建。结果字段获取这些对应的修饰值：（看下面例子好理解）

```ts twoslash
// @errors: 2341
class Params {
  constructor(
    public readonly x: number,
    protected y: number,
    private z: number
  ) {
    // No body necessary
  }
}
const a = new Params(1, 2, 3);
console.log(a.x);
//            ^?
console.log(a.z);
```

## 类表达式

<blockquote class='bg-reading'>
   <p>背景知识阅读：<br />
   <a href='https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/class'>Class expressions (MDN)</a><br/>
   </p>
</blockquote>

类表达式和类声明很类似。
唯一不同的区别是类表达式不需要名字，尽管我们可以通过它们最终绑定到的任何标识符来引用它们（比如下面赋予到someClass变量）：

```ts twoslash
const someClass = class<Type> {
  content: Type;
  constructor(value: Type) {
    this.content = value;
  }
};

const m = new someClass("Hello, world");
//    ^?
```

## `abstract` 类和成员

类，方法和TypeScript里的字段可能是 _abstract抽象的_ 。

一个 _抽象方法_ 或 _抽象字段_ 是一个值，表示还没有被实现提供。
这些成员必须在 _抽象类_ 里存在，这种抽象类不能被实例化。（只能被继承）

抽象类的角色是给子类提供基类，这个子类必须实现所有抽象成员。
当一个类没有任何抽象成员，他就是 _具体的_ 。

让我们看一个例子：

```ts twoslash
// @errors: 2511
abstract class Base {
  abstract getName(): string;

  printName() {
    console.log("Hello, " + this.getName());
  }
}

const b = new Base();
```

我们不能用`new`实例化`Base`，因为它是抽象的。
相反，我们需要创建一个派生类并实现抽象成员：

```ts twoslash
abstract class Base {
  abstract getName(): string;
  printName() {}
}
// ---cut---
class Derived extends Base {
  getName() {
    return "world";
  }
}

const d = new Derived();
d.printName();
```

注意如果我们忘记实现基类的抽象成员，我们会得到报错：

```ts twoslash
// @errors: 2515
abstract class Base {
  abstract getName(): string;
  printName() {}
}
// ---cut---
class Derived extends Base {
  // forgot to do anything
}
```

### 抽象构造签名

有时候你想接受一些类构造函数，它产生一个派生自某个抽象类的实例。

例如，你可能想要些下面代码：

（这里`ctor: typeof Base`不太理解，Base这个了类被当做了一个值？看下面的代码，有些迷糊，难道typeof ABC返回的是ABC的构造函数？）
```text
class ABC {
  name: string
  constructor(n: string) {
    this.name = n
  }
  getName() {
    return this.name
  }
}

let abc: ABC = new ABC('33')



function OOO(asd: typeof ABC) {
  const temp = new asd('333444')
  console.log(temp.getName())
}

OOO(ABC)
```

```ts twoslash
// @errors: 2511
abstract class Base {
  abstract getName(): string;
  printName() {}
}
class Derived extends Base {
  getName() {
    return "";
  }
}
// ---cut---
function greet(ctor: typeof Base) {
  const instance = new ctor();
  instance.printName();
}
```

TypeScript正确的告诉你正在实例化一个抽象类。
毕竟，给定`greet`的定义，编写这段代码完全合法的，它最终会尝试调用一个抽象类的构造函数：

```ts twoslash
declare const greet: any, Base: any;
// ---cut---
// Bad!
greet(Base);
```


相反，您想编写一个接受带有构造签名的东西的函数：（意思是下面才能正确报错）

```ts twoslash
// @errors: 2345
abstract class Base {
  abstract getName(): string;
  printName() {}
}
class Derived extends Base {
  getName() {
    return "";
  }
}
// ---cut---
function greet(ctor: new () => Base) {
  const instance = new ctor();
  instance.printName();
}
greet(Derived);
greet(Base);
```

现在TypeScript正确地告诉您可以调用哪些类函数 — `Derived`可以，因为它是具体的，但`Base`不能。

## 类之间的关系

在大多数情况下，TypeScript 中的类在结构上进行比较，与其他类型相同。

例如，下面两个类可以在对方的应用出混用，因为是相同的：

```ts twoslash
class Point1 {
  x = 0;
  y = 0;
}

class Point2 {
  x = 0;
  y = 0;
}

// OK
const p: Point1 = new Point2();
```

同样，即使没有显式继承，类之间的子类型关系也存在：

```ts twoslash
// @strict: false
class Person {
  name: string;
  age: number;
}

class Employee {
  name: string;
  age: number;
  salary: number;
}

// OK
const p: Person = new Employee();
```

这听起来很直观，但有一些案例似乎比其他案例更奇怪。（就是上面的例子，子类型关系不匹配能正常工作的奇怪例子，上面p的类型被缩小成Person了）

空类没有成员。
在结构类型系统中，没有成员的类型通常是其他任何东西的超类型。
所以如果你写一个空类（不要这样做），任何东西都可以用来代替它：

```ts twoslash
class Empty {}

function fn(x: Empty) {
  // can't do anything with 'x', so I won't
}

// All OK!
fn(window);
fn({});
fn(fn);
```
