---
layout:     post
title:      "05 - 依赖注入优于硬连接资源 "
subtitle:   "Effective-java-3rd"
date:       2020-03-17 15:30:00
author:     "linjiankai"
header-img: "img/post-bg-effective-java-3rd.png"
catalog: true
tags:
    - 学习
    - effective-java
---
## 引用底层资源的类

许多类依赖于一个或多个底层资源。例如，拼写检查器依赖于字典。我们可以将此类实现为静态工具类或者实现为单例。

``` Java
// Inappropriate use of static utility - inflexible & untestable!
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {} // Noninstantiable

    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

``` Java
// Inappropriate use of singleton - inflexible & untestable!
public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}

```

但是这两种方式都是在假设只有一本字典可用，在这个需求里面，每种语言都应该有自己的字典。改良方法可通过使`dictionary`属性设置为非`final`，然后添加一个方法来更改现有的字典，从而令其支持多个字典，但是这样编写代码容易出错，而且**无法并行工作**.因此静态工具类和单例类不适合用于需要引用底层资源的类。

## 依赖注入
在上述需求中，我们需要能够支持类的多个实例，并且每个实例都使用客户端期望的资源（多个`SpellChecker`，每个实例中都有自己的`dictionary`）。在创建新实例时将资源传递到构造器中。这种方式叫依赖项注入（dependency injection）：字典是拼写检查器的一个依赖项，当它创建时被注入到拼写检查器中。

```Java
// Dependency injection provides flexibility and testability
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```
依赖项注入可以使用任意数量的资源和任意依赖数。它保持了不变性（指的是其实例不能被修改的类，资源注入的时候已经确定了），因此多个客户端可以共享依赖对象（假设客户需要相同的底层资源）。依赖注入同样适用于构造器、静态工厂（条目1）和 builder 模式（条目2）。

当变换注入的资源，可以将资源工厂传递给构造器。工厂是指可以重复调用以创建类型实例的对象。这种称为工厂方法模式（Factory Method pattern）。

Java 8 中引入的 `Supplier<T>` 接口非常适合代表工厂。 在输入上采用 `Supplier<T>` 的方法通常应该使用有界的通配符类型（bounded wildcard type）（条目31）约束工厂的类型参数，**以便客户端能够传入一个工厂，来创建指定类型的任意子类型**。例如，下面是一个生产鑲嵌图案的方法，它利用客户端提供的工厂来生产每一个图案：

``` Java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

尽管依赖注入极大地提高了灵活性和可测试性，但它可能使大型项目变得混乱，这些项目通常包含数千个依赖项。使用依赖注入框架（如 Dagger [Dagger]、Guice [Guice] 或 Spring [Spring]）可以消除这些混乱。在们常用的spring框架里面，spring帮我们管理了可供依赖的资源，通过@Autowired[java自带的依赖注入标签为@Resource]进行资源注入），那些为手动依赖注入而设计的 API 非常适合这些框架的使用。

总而言之，不要用单例和静态工具类来实现依赖一个或多个底层资源的类，且该资源的行为会影响到该类的行为；也不要直接用这个类来创建这些资源。而应该将这些资源或者工厂传给构造器（或者静态工厂，或者`builder`模式），通过它们来创建类。这个实践就被称作依赖注入，它 **极大地提升了类的灵活性、可重用性和可测试性** 。
