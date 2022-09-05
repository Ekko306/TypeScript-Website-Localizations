---
title: The Basics
layout: docs
permalink: /zh/docs/handbook/2/basic-types.html
oneline: "Step one in learning TypeScript: The basic types."
preamble: >
  <p>欢迎来到这本手册的首页！如果这是你第一次接触TypeScript —— 你可能要从'<a href='https://www.typescriptlang.org/docs/handbook/intro.html#get-started'>Getting Started</a>'指南开始。</p>
---


JavaScript的每一个值在不同的操作下，可能都有不同的行为。听起来很抽象，看底下这个例子理解，假如我们想要在一个变量上运行`message`方法。

```js
// Accessing the property 'toLowerCase'
// on 'message' and then calling it
message.toLowerCase();

// Calling 'message'
message();
```

我们拆分上面代码理解，第一行代码去获取一个`toLowerCase`方法然后调用。第二行代码直接调用`message`。

但是假设我们不知道`message`的值是什么 —— 这个在日常情况很普遍，我们不能随便找个代码运行然后依赖它的返回值。我们对值的每个操作的行为完全取决于我们对这个值首次的定义。

- `message`是否可以被调用？
- 它是否有一个`toLowerCase`属性?
- 如果他有，`toLowerCase`是否能被调用?
- 如果他们能被调用，各自的返回值是什么?

这些问题的答案通常在我们写JavaScript代码的时候保存在我们脑子里。并且我们只能希望脑子记得的细节是正确的。

让我们像下面这样定义`message`变量。

```js
const message = "Hello World!";
```

正如你猜想的，如果我们运行`message.toLowerCase()`。我们将会得到全部小写格式的字符串。

那第二行代码呢？如果你熟悉JavaScript，你将会知道这个会抛出异常报错：

```txt
TypeError: message is not a function
```

如果我们避免这样的错误就好了。

当我们运行我们的代码的时候，JavaScript运行时需要先分析出所有值的类型（值的行为和能力），然后再去执行不同的操作。这个就是上面报错`TypeError`指出，`"Hello World!"`字符串不能被当做函数调用。


对于一些值，例如原生的`string`和`number`，我们可以通过typeof运损区分他的运行时类型。但是对于哪些函数，没有运行时的语法来区分他们的类型，比如下面这个函数：

```js
function fn(x) {
  return x.flip();
}
```

我们可以通过阅读函数代码 _观察到_ 我们之后传入一个带有`flip`属性的对象，函数才能正常运行，但是 JavaScript 并没有我们可以在代码运行时检查的方式提示这些信息。唯一的方式是在纯净的JavaScript传入特定的值尝试运行一下，看结果怎么样。这种JS的特性让你很难在代码运行之前知道将要工作的流程，意味着你写代码的时候更难代码的结果。

这样看， _Js的类型_ 是描述那些值会传递给`fn`以及哪些会崩溃的概念。JavaScript只会相信 _动态类型_ （只能运行代码才能了解）。

另一个可能的方法就是使用静态类型系统，在代码运行前预测跑起来的行为。

## 静态类型检测

考虑我们将一个 `string` 当做函数运行得到的 `TypeError`。
_大多数人_ 不想在运行代码的时候得到这些错误 —— 他们被认为是bug！
并且当我们写新代码的时候，我们尽力避免新bug。

如果我们只是加一点代码，保存我们的文件，从新运行代码，然后立即看到报错，我们可能可以迅速发现问题；但事实不总是这样。
我们可能没有把功能测试完全，所以我们可能永远不会真正遇到可能抛出的潜在错误！
或者如果我们足够幸运目睹了这个潜在的错误，我们可能最终要进行大规模的重构并添加许多我们不得不挖掘的不同代码。

理想情况下，我们 _代码运行前_ 用工具帮我们发现这些漏洞。
那就是像TypeScript这种静态类型检查器做的。
_静态类型系统_ 描述了我们的值在项目运行时的类型和行为。
一个像TypeScript的类型检查器使用这些信息告诉我们什么时候事情可能会出轨。
```ts twoslash
// @errors: 2349
const message = "hello!";

message();
```

在我们首先运行代码之前，使用 TypeScript 运行最后一个示例会给我们一个错误消息。

## 非异常错误

目前为止我们已经讨论了一些类似运行时的错误 — JavaScript运行时告诉我们它认为某些事很荒谬的情况。
哪些情况出现的原因是[ECMAScript规范](https://tc39.github.io/ecma262/)有关于语言在遇到意外情况时应该如何表现的明确说明。

例如，这个规范说尝试在一个不能调用的东西上调用方法会报错。（比如上面一个字符串当做函数调用）
可能哪些听起来是“明显的错误”，并且你可以想象访问对象上不存在的属性也会引发错误。
但是，JavaScript给我们不同的行为，没有报错，而是返回`undefined`：


```js
const user = {
  name: "Daniel",
  age: 26,
};

user.location; // returns undefined
```

最终，TypeScript静态类型系统必须告诉在这个系统这些应该被标记为错误，及时它是不会立即抛出错误的“有效”的JavaScript。
在TypeScript里，下面的代码会报错，告诉`location`没有被定义：

```ts twoslash
// @errors: 2339
const user = {
  name: "Daniel",
  age: 26,
};

user.location;
```

虽然有时这意味着您可以表达的内容上进行权衡（可能有时候允许undefined），但其目的是捕捉我们程序中的合法错误。
并且TypeScript捕获了很多合法的错误。

例如：拼写错误，

```ts twoslash
// @noErrors
const announcement = "Hello World!";

// How quickly can you spot the typos?
announcement.toLocaleLowercase();
announcement.toLocalLowerCase();

// We probably meant to write this...
announcement.toLocaleLowerCase();
```

没有调用的函数，

```ts twoslash
// @noUnusedLocals
// @errors: 2365
function flipCoin() {
  // Meant to be Math.random()
  return Math.random < 0.5;
}
```

或者基本逻辑错误，

```ts twoslash
// @errors: 2367
const value = Math.random() < 0.5 ? "a" : "b";
if (value !== "a") {
  // ...
} else if (value === "b") {
  // Oops, unreachable
}
```

## 使用类型的工具

TypeScript可以在我们代码里有错误的时候捕捉bug。
很好，但是TypeScript _也_ 可以防止我们在写代码的第一个位置防止我们制造错误。

类型检查器提供一些信息，比如我们是否正在访问变量和其他属性的正确属性之类的信息。
一旦有这些信息，它还能开始 _建议_ 哪些属性是你想要用的。

意味着TypeScript也可以用于编辑代码，并且核心类型检查器可以提供错误信息并且在你编辑器里提供类型补全。这就是人们经常谈论的把TypeScript作为辅助编写的工具。

<!-- prettier-ignore -->
```ts twoslash
// @noErrors
// @esModuleInterop
import express from "express";
const app = express();

app.get("/", function (req, res) {
  res.sen
//       ^|
});

app.listen(3000);
```

TypeScript非常重视工具，并且超粗了键入时的补全和错误。
一个支持TypeScript的编辑器可以提供“快速修复”来自动修复错误，重构以轻松重新组织代码，以及跳转到变量定义，或者找到给定变量的所有引用的等有用的导航功能。
上面这些功能都是基于类型检查器并且是跨平台的，所以很可能[你最喜欢的编辑器有 TypeScript 支持](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Editor-Support)。

## `tsc`, TypeScript编译器

我们已经讨论了类型检查，但是我们还没有使用过类型检查器，
让我们熟悉下我们的新朋友`tsx`，TypeScript编译器。
首先我们通过npm获取他。

```sh
npm install -g typescript
```

> 这个代码会全局安装`tsx`
> 如果你想从本地`node_modules`包运行`tsx`，你可以用`npx`或者相似的工具。

现在让我们新建一个空文件夹，并且写我们第一个TypeScript项目：
`hello.ts`：

```ts twoslash
// Greets the world.
console.log("Hello world!");
```

注意，这里没有多余的ts装饰，这个"hello world"程序看起来和你用JavaScript编写的"hello world"程序一样。
并且现在让我们通过`tsc`命令执行类型检查，它在我们安装`typescript`包的时候已经被安装了。

```sh
tsc hello.ts
```

Tada!

等等，“tada”到底是什么？
我们运行了`tsc`但是什么都没发生。
嗯。。。没有类型报错，所以我们没有在控制台得到任何输出，因为没有任何要报告的。.

但是再检查看看，我们有一些 _文件_ 输出。
如果我们检查当前的文件夹，我们会在`hello.ts`旁边看到`hello.js`文件。
那个是`hello.ts`文件在`tsc` _编译_ 和 _转换_ 后输出的扁平JavaScript文件。
并且如果我们检查内容，我们会发现TypeScript在处理`.ts`文件后会输出什么：

```js
// Greets the world.
console.log("Hello world!");
```
在这种情况下，只有很少部分需要TypeScript转换，所以他和我们写的ts代码几乎一样。
编译器尝试转换输出像是人写出来的简洁可读的代码。
虽然这并不总是那么容易，但 TypeScript 会始终如一地分析代码并缩进，并在编译过程中注意我们的代码何时跨越不同的代码行，并试图保留注释。

如果我们 _故意_ 实现一个类型检查错误呢？
让我们重写`hello.ts`：

```ts twoslash
// @noErrors
// This is an industrial-grade general-purpose greeter function:
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date}!`);
}

greet("Brendan");
```

如果我们再次运行`tsc hello.ts`，注意我们在命令行上得到了新的错误！

```txt
Expected 2 arguments, but got 1.
```

TypeScript正在告诉我们我们忘记传递一个参数给`greet`函数，这是理所当然的。
目前位置我们仅仅写了标准的JavaScript代码，并且类型检查仍然能够检查我们代码的问题（没有加上ts语法哪些东西）。
感谢TypeScript！

## 带着错误输出

你应该注意到的一件事是在最后上一个例子我们的`hello.js`文件被再次修改了。
如果我们再次打开文件我们会发现内容基本和我们的输入文件类似。
当你看到`tsc`对我们的代码报告错误的时候你可能很吃惊，但是这个是基于TypeScript的一个核心价值：再多花些时间，_你_ 会更了解TypeScript。

重申一下，类型检查代码限制了你可以运行的程序种类（就是有些规则你你可以决定类型检查器检不检查），因此需要权衡类型检查器认为可接受的类型。
大多数时候没关系，但是有些情况下，这些检查会妨碍你。
例如，假设你从JavaScript代码迁移到TypeScript并且有TypeScript报错。
最终你可能会不停的修改来满足类型检查器，但是原始的JavaScript代码已经能工作了！
为什么你迁移到TypeScript，但是TypeScript要阻止你？（原本可以工作）。

所有TypeScript不应该当着你的路。
当然，随着时间，你可能想要希望更加好的防御错误，并使TypeScript的行为更加严格。
那种情况，你可以使用[`noEmitOnError`](/tsconfig#noEmitOnError) 编译器选项。
尝试修改你的`hello.ts`文件并且带着标志参数运行`tsc`（上面缺少参数的代码，可以js正常运行输出的）：

```sh
tsc --noEmitOnError hello.ts
```

你将会看到`hello.js`不会更新。

## 显式类型

直到现在，我们还没有告诉TypeScript`person`或者`date`的类型。
让我们来编辑代码告诉TypeScript`person`是`string`类型，并且`date`应该是一个`Date`对象。
我们还会在`date`上调用`toDateString()`方法。

```ts twoslash
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
```

我们做的是在`person`和`date`上增加 _类型注释_ 来描述`greet`调用需要的参数。
你可以阅读这个信息看做"`greet`函数接收一个`string`类型的`person`参数，和一个`Date`类型的`date`参数"。

有了这个，TypeScript可以在`greet`被错误调用的时候报错其他情况。
例如...

```ts twoslash
// @errors: 2345
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

greet("Maddison", Date());
```

嗯哼？
TypeScript给第二个参数报错了，但是为什么？

可能令人吃惊的，在JavaScript里调用`Date()`返回一个`string`类型。
换个方式看，用`new Date()`来替代`Date()`构建`Date`作为入参会得到预期的。

不管怎么样，我们可以快速修复这个错误：

```ts twoslash {4}
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

greet("Maddison", new Date());
```

时刻注意，我们不需要总是写显式的类型注释。
很多情况下，TypeScript甚至可以为我们 _推断_ （或者找出）类型，即使我们忽略他。（那就简单的逻辑不管类型，先写代码再ide infer出来自动补全（webstorm可以））。

```ts twoslash
let msg = "hello there!";
//  ^?
```

即使我们没有告诉TypeScript`msg`是`string`类型，它还能够判断出来。
这是一个特性，当类型系统最终会推断出相同的类型时，最好不要添加注释。（一方面是避免画蛇添足，也能代码简洁。）

> 注意：如果您将鼠标悬停在该单词上，上面代码例子中的消息气泡就是的编辑器将会显示的内容。

## 擦除类型

让我们看看当我们对上面的`greet`函数用`tsc`编译之后的JavaScript：

```ts twoslash
// @showEmit
// @target: es5
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

greet("Maddison", new Date());
```

注意两件事情：

1. 我们`person`和`date`参数不再有类型注释了
2. 我们的`字符串模板` — 那个使用`` ` ``的反引号字符串 — 被转换为带有连接的纯字符串。

之后我们会更多的讲第二点，你现在我们先关注第一点。
类型注释不是JavaScript的一部分（或者说ECMAScript标准是迂腐的(๑¯∀¯๑)），所以真的没有任何浏览器或其他运行时可以在未经修改的情况下运行TypeScript。
大多数特定于TypeScript的代码都被删除了，同样地，我们的类型注释也被完全删除了。

> **请记住**：类型注释永远不会改变你项目的运行时行为。（实际运行被删除了？）

## 降级

上面一个其他另外的不同之处是我们的模板字符串从

```js
`Hello ${person}, today is ${date.toDateString()}!`;
```

转换为了

```js
"Hello " + person + ", today is " + date.toDateString() + "!";
```

为什么和这个会发生？

模板字符串是一个叫做ECMAScript 2015版本的ECMAScript标准（也叫作ECMAScript 6，ES2015，ES6等 — _别问了_）。
TypeScript也有一些能力从ECMAScript的新版本重写回更早期的ECMAScript 3或者 ECMAScript 5的版本代码（也叫作ES3和ES5）。
这个从更新或者“更高”版本的ECMAScript转换为更老或者“更低”版本的过程有时候叫做 _降级_。

默认TypeScript的目标是ES3版本，一个ECMAScript很老的版本。
我们可能可以通过使用[`target`](/tsconfig#target)选项来选择更新的版本。
带着`--target es2015`运行会将TypeScript转换为目标ECMAScript 2015，意味着代码应该更在支持ECMAScript 2015的环境中运行。
所以运行`tsc --target es2015 hello.ts`让我们有如下输出


```js
function greet(person, date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}
greet("Maddison", new Date());
```
> 虽然默认目标是ES3，但是现在浏览器的主流都支持ES2015.
> 因此大多数开发者可以安全的使用ES2015或者更高的语法作为目标（在tsconfig里配置target），除非与某些古代浏览器的兼容性很重要。

## 严格模式

不同的用户使用TypeScript是为了类型检查器的不同特性。
有些人会寻找一种宽松的选择加入体验，它可以帮助验证他们程序的某些部分，并且作为体合适的工具。
这个就是TypeScript的默认配置，类型是可选的，推理采用最宽松的类型，并且不会检查潜在的`null`/`undefined`（默认规则允许）。
较像`tsc`在遇到错误报错，但是还是编译出js文件一样，这些默认设置不会妨碍你。
如果你从现有的JavaScript迭代，默认配置是理想的异步。

但相反的是，很多用户更喜欢让TypeScript尽可能多地立即验证，这就是TypeScript还提供了严格模式的原因。
这些严格设置将静态类型检查从开关（是否检查你的代码）变成更接近拨号的东西。（就是很多严格规则是false，需要手动开启）
你开启的开关越多，TypeScript越会给你做更多检查。
这需要一点额外的工作，但是一般从长远来看，它会值得付出，并且可以进行更彻底的检查和成为更准确的工具。
尽可能地，一个新的代码库应该始终开启这些严格性检查。

TypeScript有几种不同的类型检查严格性标志可以开关，并且所有我们的例子都是在他们全部打开的情况下除非有特殊说明。在CLI的[`strict`](/tsconfig#strict)标志，或者在[`tsconfig.json`](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)里设置`"strict": true`会同事打开所有规则，并且自己可以选择性关闭一些。
两个你最应该了解的是[`noImplicitAny`](/tsconfig#noImplicitAny) 和 [`strictNullChecks`](/tsconfig#strictNullChecks)。

## `noImplicitAny`

回想一下，在某些地方，TypeScript没有尝试给我们推断类型并且，却是回退到最宽松的类型：`any`。
这并不是坏的 — 因为回退到`any`变成了最简单的JavaScript体验。

但是，使用`any`通常会第一时间破坏你使用TypeScript的目的。
你项目里的类型越多，你使用TypeScript后获得的验证和工具就越多，就意味着你编写代码遇到的错误越少。
开启[`noImplicitAny`](/tsconfig#noImplicitAny)选项将会在任何类型隐式推断为`any`的时候报错。

## `strictNullChecks`

默认情况下，像`null`和`undefined`这些值可以被赋予任何其他值。
这个可能让代码更简单，但是忘记处理`null`和`undefined`石造成世界上无尽的bug的原因—可以看例子 [十亿美元的错误](https://www.youtube.com/watch?v=ybrQvs4x0Ps)!
[`strictNullChecks`](/tsconfig#strictNullChecks)选项可以让处理`null`和`undfined`更明显，并且 _让我们_ 不必担心我们是否 _忘记_ 处理`null`和`undefined`。
