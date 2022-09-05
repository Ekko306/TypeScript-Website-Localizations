---
title: Creating Types from Types
layout: docs
permalink: /zh/docs/handbook/2/types-from-types.html
oneline: "An overview of the ways in which you can create more types from existing types."
---

TypeScript类型系统非常强大，因为它允许 _根据其他类型_ 表达类型。

这个思想的最简单形式就是泛型，我们实际上有各种各样的 _类型运算符_ 可供使用。
也可以用我们已经拥有的 _值_ 来表达类型。（通过变量表达类型）

通过合并不同的类型操作，我们可以以简洁、可维护的方式表达复杂的操作和值。
在这一节，我们将介绍根据现有类型或值来表达新类型的方法。

- [Generics](/zh/docs/handbook/2/generics.html) - 可以传入参数的类型
- [Keyof Type Operator](/zh/docs/handbook/2/keyof-types.html) - 使用`keyof`操作符来创建新类型
- [Typeof Type Operator](/zh/docs/handbook/2/typeof-types.html) - 使用`typeof`操作符来创建新类型
- [Indexed Access Types](/zh/docs/handbook/2/indexed-access-types.html) - 使用`Type['a']`语法来获取一个元素的子集
- [Conditional Types](/zh/docs/handbook/2/conditional-types.html) - 在类型系统里像if声明一样的类型
- [Mapped Types](/zh/docs/handbook/2/mapped-types.html) - 通过映射现有类型中的每个属性来创建类型
- [Template Literal Types](/zh/docs/handbook/2/template-literal-types.html) - 通过模板文字字符串更改属性的映射类型
