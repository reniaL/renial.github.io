---
layout: post
title:  "静态类型语言 VS. 动态类型语言"
tags: code
---

Java's verbosity made us all hate type systems in the early 2000s so many of us migrated to dynamic languages such as Python, Ruby in the mid 2000s that allowed us to work fast and loose and get things done.

After about 10 years of coding in a fit of passion we ended up with huge monolithic projects written in dynamic languages that where extremely brittle.

> 21世纪初，Java 的冗余让我们讨厌静态类型系统，然后纷纷转向动态类型语言 (Python, Ruby)，以求快速完成工作。这样的情况持续了10多年之后，我们用动态语言造出了不少难以维护的、大型的单体项目。

**bunderbunder**

Java's type system mostly just gets in the way.

Then, to add insult to injury, its static type system is very weak. A combination of type erasure, a failure to unify arrays with generic types, and poor type checking in the reflection system means that Java's static typing doesn't give you particularly much help with writing well-factored code compared to most other modern high-level languages.

It's statically typed, but with basically no type inference (the diamond operator is something, I guess, but not much). So you end up putting a lot of time into manually managing types. That creates friction when writing code, since you need to remember the name of the return type of every method you call in order to keep the compiler happy. Worse, it creates friction when refactoring code, since any change that involves splitting or merging two types, or tweaking a method's return type, ends up forcing a multitude of edits to other files in order to propitiate the compiler. I've seen 100-file pull requests where only one of those file changes was actually interesting.

> 说了 Java 类型系统的缺点。Java 是静态类型的，但基本上没有类型推断（？），我们得记住调用的每个方法的返回类型。重构也很蛋疼，有时仅仅修改一个类型，就得更新全部引用了这块代码的文件（嗯，我反而觉得有好处。到底是好是坏？）。
> 还说，Java 的静态类型系统很弱，由于 type erasure、泛型、反射等特性的结合，导致了这个结果。