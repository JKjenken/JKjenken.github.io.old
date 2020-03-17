---
layout:     post
title:      "04 - 使用私有构造器执行非实例化 "
subtitle:   "Effective-java-3rd"
date:       2020-03-17 09:50:00
author:     "linjiankai"
header-img: "img/post-bg-effective-java-3rd.png"
catalog: true
tags:
    - 学习
    - effective-java
---
## 非实例化的某些类

当我们需要写一个只包含静态方法和静态字段的类时（有些人滥用这些类从而避免以面向对象方式思考从而编写过程化的程序，但是它们确实有着特殊的用途），从java8开始，也可以将这些方法放在自己编写的接口中，这样的类可以用于在 final 类上对方法进行分组，因为不能将它们放在子类中。例如，按照  `java.lang.Math` 或 `java.util.Arrays` 的方式，把基本类型的值或数组类型上的相关方法组织起来;或者通过`java.util.Collections` 的方式，把实现特定接口上面的静态方法进行分组，也包括工厂方法。

针对这样的工具类(utility classes)，其本身不是设计用来被实例化的，因为实例化对它没有任何意义。然而，在没有显式构造器的情况下，编译器提供了一个公共的、无参的默认构造器。对于用户来说，该构造器与其他构造器没有什么区别。在已发布的 API 中经常看到无意识的被实例的类。

错误的方式是试图通过创建抽象类来强制执行非实例化，这是行不通的。因为该类可以被子类化，并且子类可以被实例化。正确的方法是通过包含一个私有构造器来实现类的非实例化。

``` Java
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    // or Suppresses default constructor, ensuring non-instantiability.
    private UtilityClass() {
        throw new AssertionError();
    }
    ... // Remainder omitted
}
```

由于该构造器给人的感觉好像构造器就是设计成不能调用的一样。因此，添加注释是种明智的做法。

一点小小的副作用在于这样的类不能子类化。所有的构造器都必须显式或隐式地调用父类构造器，而在这种情况下子类则没有可访问的父类构造器来调用。
