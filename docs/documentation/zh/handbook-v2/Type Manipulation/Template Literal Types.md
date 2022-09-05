---
title: Template Literal Types
layout: docs
permalink: /zh/docs/handbook/2/template-literal-types.html
oneline: "Generating mapping types which change properties via template literal strings."
---

模板文字类型建立在[字符串文字类型](/docs/handbook/2/everyday-types.html#literal-types)之上，并且能够通过联合类型扩展成许多字符串。

他们和[JavaScript里的模板字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)有相同的语法，但是是使用在类型场合。
当与具体文字类型一起使用时，一个模板文字同构连接内容生产新的字符串文字类型。

```ts twoslash
type World = "world";

type Greeting = `hello ${World}`;
//   ^?
```

当一个联合类型被用在插入值的地方，类型变成每个联合成员的每个可能字符串文字的集合：

```ts twoslash
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
//   ^?
```

对于模板文字类型里插入值的地方，如果都是联合类型会造成交叉乘积：

```ts twoslash
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";
// ---cut---
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
type Lang = "en" | "ja" | "pt";

type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`;
//   ^?
```

我们通常建议人们对大型字符串联合使用提前生成（就是太大的交叉集合性能有问题），但在小集合的情况很有用。

### 类型的字符串联合类型

当基于类型内信息定义一个新字符串时，模板文字的强大之处就体现了。

考虑一个函数（`makeWatchedObject`），这个函数把一个叫做`on()`的函数添加到一个传递的对象。在JavaScript里，他的调用可能看起来像`makeWatchedObject(baseObject)`。我们可以想想基础的对象如下所示：

```ts twoslash
// @noErrors
const passedObject = {
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26,
};
```

`on`函数将会被添加到基础对象上去，并且需要两个参数，`eventName: string`和`callBack: Function`。

`eventName`应该是`attributeInThePassedObject + "Changed"`的形式，因此`firstNameChanged`会是从`firstName`的基础对象里派生出来的。

`callBack`函数，当被调用的时候： 
 * 应该传入和名称`attributeInThePassedObject`关联类型的值；因此由于`firstName`类型是`string`，`firstNameChanged`事件的回调期望在调用时将`string`传递给它，同样，与`age`相关的事件应该使用`number`类型的参数调用
 * 应该有`void`返回类型（讲解例子简化）

原生的`on()`函数签名因此会是`on(eventName: string, callBack: (newValue: any) => void)`的样子。然而，在前面的描述中，我们确定了希望在代码中记录的重要类型约束（就是`eventName`的`string`应该被传递过来成我们约束的特有的Changed啥的）。模板文字类型让我们将这些约束带入我们的代码中。（这种就是利用模板文字来约束参数具体值）

```ts twoslash
// @noErrors
declare function makeWatchedObject(obj: any): any;
// ---cut---
const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26,
});

// makeWatchedObject has added `on` to the anonymous Object

person.on("firstNameChanged", (newValue) => {
  console.log(`firstName was changed to ${newValue}!`);
});
```

注意`on`是监听`"firstNameChanged"`事件，不仅是`"firstName"`。如果我们能确保符合条件的事件的名称是和我们内部的attribute名字加上"Changed"，我们的`on()`原生实现就可以更加健壮的。当我们在JavaScript做这种操作时会很舒适，例如``Object.keys(passedObject).map(x => `${x}Changed`)``。在 _类型系统里_ 的模板字符串也提供了相似的方法进行字符串操作：

```ts twoslash
type PropEventSource<Type> = {
    on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void;
};

/// Create a "watched object" with an 'on' method
/// so that you can watch for changes to properties.
declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;
```

有了上述实现，我们可以在传入了错误的属性时候报错（很强，可以用代码就表达你的逻辑，但是复杂度也提升了）：

```ts twoslash
// @errors: 2345
type PropEventSource<Type> = {
    on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void;
};

declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>;
// ---cut---
const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});

person.on("firstNameChanged", () => {});

// Prevent easy human error (using the key instead of the event name)
person.on("firstName", () => {});

// It's typo-resistant
person.on("frstNameChanged", () => {});
```

### 在模板文字里推断

注意我们还没有获取到原生传递类型的所有信息。给一个`firstName`事件（例如`firstNameChanged`事件)，我们应该希望回调将会接收一个`string`的参数。类似的，对于`age`的回调应该接收一个`number`的参数。我们原来用`any`来表示`callBack`参数的类型。同样，模板文字类型可以确保属性的数据类型与该属性的回调的第一个 参数类型相同。

使这个成为可能的关键见解是：我们可以使用具有**泛型**的函数，这样之后（下面的话说的迷迷糊糊的，直接看代码最明白，就是`Type[Key]`）：

1. 第一个参数中使用的字面量被捕获为字面量类型。
2. 该文字类型可以在泛型中的有效属性联合部分被验证。
3. 可以使用索引访问在在泛型结构中查找已验证属性的类型。
4. 可以 _然后_ 应用此类型信息来确保参数回调属于统一类型。

```ts twoslash
type PropEventSource<Type> = {
    on<Key extends string & keyof Type>
        (eventName: `${Key}Changed`, callback: (newValue: Type[Key]) => void ): void;
};

declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;

const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});

person.on("firstNameChanged", newName => {
    //                        ^?
    console.log(`new name is ${newName.toUpperCase()}`);
});

person.on("ageChanged", newAge => {
    //                  ^?
    if (newAge < 0) {
        console.warn("warning! negative age");
    }
})
```

这里我们把`on`转换为了泛型方法。

当一个用户使用`firstNameChanged`调用时，TypeScript会尝试获取`Key`的正确类型。
为了实现这个，它会将`Key`与`Changed`之前的内容进行匹配，并推断出字符串`"firstName"`。
一旦TypeScript发现了这一点，`on`方法可以获取原始对象上的`firstName`类型，在本例中为`string`。
相似的，当使用`ageChanged`调用时，TypeScript会找到属性`age`的类型，即`number`。

推断可以用不同的方式组合，通常是解构字符串，并以不同的方式重新组合他们。

## 内在字符串操作类型

为了帮助操作字符串操作，TypeScript包含了一些类型集合可以在字符串操作里使用。这些类型是编译器内置的，但是不能在TypeScript的`.d.ts`里发现。

### `Uppercase<StringType>`

将每个字符串的字符变成大写版本。

##### Example

```ts twoslash
type Greeting = "Hello, world"
type ShoutyGreeting = Uppercase<Greeting>
//   ^?

type ASCIICacheKey<Str extends string> = `ID-${Uppercase<Str>}`
type MainID = ASCIICacheKey<"my_app">
//   ^?
```

### `Lowercase<StringType>`

将每个字符串的字符变成小写版本。

##### Example

```ts twoslash
type Greeting = "Hello, world"
type QuietGreeting = Lowercase<Greeting>
//   ^?

type ASCIICacheKey<Str extends string> = `id-${Lowercase<Str>}`
type MainID = ASCIICacheKey<"MY_APP">
//   ^?
```

### `Capitalize<StringType>`

将字符串的第一个字符变成大写。

##### Example

```ts twoslash
type LowercaseGreeting = "hello, world";
type Greeting = Capitalize<LowercaseGreeting>;
//   ^?
```

### `Uncapitalize<StringType>`

将字符串的第一个字符变成小写。

##### Example

```ts twoslash
type UppercaseGreeting = "HELLO WORLD";
type UncomfortableGreeting = Uncapitalize<UppercaseGreeting>;
//   ^?
```

<details>
    <summary>内在字符串操作类型的技术类型</summary>
    <p>从 TypeScript 4.1 开始，这些内部函数的代码直接使用 JavaScript 字符串运行时函数进行操作，并且不能在语言环境知道（？不知道咋理解）</p>
    <code><pre>
function applyStringMapping(symbol: Symbol, str: string) {
    switch (intrinsicTypeKinds.get(symbol.escapedName as string)) {
        case IntrinsicTypeKind.Uppercase: return str.toUpperCase();
        case IntrinsicTypeKind.Lowercase: return str.toLowerCase();
        case IntrinsicTypeKind.Capitalize: return str.charAt(0).toUpperCase() + str.slice(1);
        case IntrinsicTypeKind.Uncapitalize: return str.charAt(0).toLowerCase() + str.slice(1);
    }
    return str;
}</pre></code>
</details>
