---
title: The TypeScript Handbook
layout: docs
permalink: /zh/docs/handbook/intro.html
oneline: Your first step to learn TypeScript
handbook: "true"
---

## 关于这本手册

自从JavaScript被介绍给编程社区已经超过了20年，现在它已经成为最被广泛应用在跨平台系统的语言之一。从一个最初发明为了给页面添加琐碎交互的脚本语言起步，JavaScript 已发展成为各种规模的前端和后端应用程序的首选语言。与此同时用JavaScript编程的项目的大小，作用域和复杂度成倍的增长，然而JavaScript语言的表达代码不同模块之间的联系的能力并没有增长。同时JavaScript还有一些相当奇怪的运行时语义，这种语言和项目复杂的之间的不匹配已经让JavaScript开发成为一项难以大规模管理的任务。

程序员最容易犯的几种错误可以被归纳为类型错误：在预期不同类型的值的情况下使用了某种类型的值，这个可能是简单的拼写错误，对库接口的错误理解，运行时行为的不正确假设，或者其他错误。TypeScript的目标是为JavaScript程序提供一种静态类型检测 —— 换句话所，是在你代码运行之前的一个（静态）工具，这个工具保证你的项目的类型是正确的（类型检测）。

如果你没有JavaScript就来使用TypeScript，并想把TypeScript作为你的第一门语言，我满建议你读下面两个文档，要么是[Microsoft Learn JavaScript tutorial](https://docs.microsoft.com/javascript/)，要么是[JavaScript at the Mozilla Web Docs](https://developer.mozilla.org/docs/Web/JavaScript/Guide)。
如果你已经使用过其他语言，你应该在读完手册后很快适应JavaScript的语法。

## 这个手册的结构是什么样子

这个手册被拆分成两部分：

- **手册**

  TypeScript手册想要为日常的程序提供一个成为解释 TypeScript 的综合文档。你可以从左边的导航栏从头到尾的阅读。

  你应该希望每一章讲解的概念给你深刻的理解。  但是TypeScript手册不是一个完整的规范（不能面面俱到），它只是想成为TypeScript所有特性和行为的综合指南。

  阅读完整个手册的同学应该能够：

  - 阅读和理解日常使用的TypeScript语法和模式。
  - 对重要的ts编译选项的影响给出解释。
  - 在大多数场景正确预测出类型系统的行为。

  为了清晰和简洁期间，手册的主要内容不会探索所涵盖功能的所有边缘情况或细节。你可以在“reference参考”的文章里找到特定概念的更多细节。

- **参考文件**

  导航栏的手册下面的参考章节是为了给TypeScript的某个特定运行细节提供丰富的知识。你可以从头到尾的阅读，但是每个章节给某个概念提供更深的解释 —— 意味着章节之间没有连续性。

### 次要目标

这个手册也是想成为一个简要的文档，能够在几个小时就能被阅读完。因此有些概念可能为了保持简短不会被全部介绍。

特别说明的，这个手册也不会对JavaScript基础里的函数、类或者闭包有全面的介绍。在适当情况下，我们会提供一些背景阅读的链接，让你来补足这些概念。

这个手册也不想成为一个语言规范的替代。在某些情况下，一个行为的边界条件或者正式描述可能会被一种高概括、易于理解的方式略过。和这个相反的是，有一些参考页面将会有对某些TypeScript行为的精确和特别详细的介绍。参考页面不适合给不熟悉TypeScript的人阅读，所以他们可能使用一些高级的术语或者参考概念，你可能从来没听过。

最后，这个手册也不会包含TypeScript和其他工具的交互，除非有些必要的情况。有些像如何在webpack, rollup, parcel, react, babel, closure, lerna, rush, bazel, preact, vue, angular, svelte, jquery, yarn, 或者 npm里配置TypeScript的方法也不会涉及。你可以在网上的其他地方找到这些资源。

## 起步

在你开始[The Basics](/docs/handbook/2/basic-types.html)之前，我们推荐你读一下下面的介绍页。这些介绍是为了将TypeScript和你最喜欢的编程语言之间的相似性和区别强调出来，并且消除一些对于这些语言的误区。

- [TypeScript for New Programmers](/docs/handbook/typescript-from-scratch.html)
- [TypeScript for JavaScript Programmers](/docs/handbook/typescript-in-5-minutes.html)
- [TypeScript for OOP Programmers](/docs/handbook/typescript-in-5-minutes-oop.html)
- [TypeScript for Functional Programmers](/docs/handbook/typescript-in-5-minutes-func.html)

否则，请跳转到[The Basics](/docs/handbook/2/basic-types.html)或者获取[Epub](/assets/typescript-handbook.epub) 与 [PDF](/assets/typescript-handbook.pdf)的副本。
