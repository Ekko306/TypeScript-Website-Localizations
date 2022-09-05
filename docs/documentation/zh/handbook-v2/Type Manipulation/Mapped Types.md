---
title: Mapped Types
layout: docs
permalink: /zh/docs/handbook/2/mapped-types.html
oneline: "Generating types by re-using an existing type."
---

当你不想自己重复一些操作时，有时一种类型需要基于另一种类型。

映射类型建立在索引签名的语法之上，用于声明未提前声明的属性类型：
```ts twoslash
type Horse = {};
// ---cut---
type OnlyBoolsAndHorses = {
  [key: string]: boolean | Horse;
};

const conforms: OnlyBoolsAndHorses = {
  del: true,
  rodney: false,
};
```

一个映射类型是一个泛型类型，它传入一个联合类型的`PropertyKey`（通常 [通过 `keyof`](/docs/handbook/2/indexed-access-types.html)创建），然后通过迭代键值来创建类型：

```ts twoslash
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
```

在这个例子里，`OptionsFlags`将会接收`Type`类型的所有属性，然后把值转换为布尔值。

```ts twoslash
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
// ---cut---
type FeatureFlags = {
  darkMode: () => void;
  newUserProfile: () => void;
};

type FeatureOptions = OptionsFlags<FeatureFlags>;
//   ^?
```

### 映射修改器

有两个额外的修改器可以在映射时使用：`readonly`和`?`，他们相应的影响可修改性和可选性。

你可以在上面两个修改器前缀使用`-`或者`+`来删除或者增加修改器。如果你没有写前缀，默认是`+`。

```ts twoslash
// Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
  readonly id: string;
  readonly name: string;
};

type UnlockedAccount = CreateMutable<LockedAccount>;
//   ^?
```

```ts twoslash
// Removes 'optional' attributes from a type's properties
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};

type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};

type User = Concrete<MaybeUser>;
//   ^?
```

## 通过`as`重新映射键

在TypeScript4.1以及更高版本中，您可以使用映射类型中的`as`子句重新映射映射类型中的键：

```ts
type MappedTypeWithNewProperties<Type> = {
    [Properties in keyof Type as NewKeyType]: Type[Properties]
}
```

你可以利用[模板文字类型](/docs/handbook/2/template-literal-types.html)之类的功能从之前的属性名称中创建新的属性名称：

```ts twoslash
type Getters<Type> = {
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};

interface Person {
    name: string;
    age: number;
    location: string;
}

type LazyPerson = Getters<Person>;
//   ^?
```

你可以通过条件类型生成`never`来过滤掉键（下面Exclude之后返回never，`as never`就是丢掉键）：

```ts twoslash
// Remove the 'kind' property
type RemoveKindField<Type> = {
    [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
};

interface Circle {
    kind: "circle";
    radius: number;
}

type KindlessCircle = RemoveKindField<Circle>;
//   ^?
```

你可以映射任意联合类型，不仅仅是`string | number | symbol`的联合，而是任意类型的联合 （下面映射的就是SquarEvent和CircleEvent类型别名对象，然后键值取kind属性）：

```ts twoslash
type EventConfig<Events extends { kind: string }> = {
    [E in Events as E["kind"]]: (event: E) => void;
}

type SquareEvent = { kind: "square", x: number, y: number };
type CircleEvent = { kind: "circle", radius: number };

type Config = EventConfig<SquareEvent | CircleEvent>
//   ^?
```

### 进一步探索

映射类型与此类型操作部分（这个Type Manipulation合集）中的其他功能能很好的配合使用，例如这里是映射类型和[条件类型](/docs/handbook/2/conditional-types.html)配合，根据对象是否有`pii`是`true`来决定把值是否转换为`true`或`false`：

```ts twoslash
type ExtractPII<Type> = {
  [Property in keyof Type]: Type[Property] extends { pii: true } ? true : false;
};

type DBFields = {
  id: { format: "incrementing" };
  name: { type: string; pii: true };
};

type ObjectsNeedingGDPRDeletion = ExtractPII<DBFields>;
//   ^?
```
