---
layout:     post
title:      "06 - 避免创建不必要对象 "
subtitle:   "Effective-java-3rd"
date:       2020-03-17 15:30:00
author:     "linjiankai"
header-img: "img/post-bg-effective-java-3rd.png"
catalog: true
tags:
    - 学习
    - effective-java
---
## 不必要对象
``` Java
String s = new String("Duplicate");  // DON'T DO THIS!
```

语句在每次执行时都会创建一个新的String实例，这些对象的创建不是必须的。String 构造方法 `Duplicate` 的参数本身就是一个 `Duplicate` 实例，它与构造方法创建的所有对象的功能相同。可以改进为：

``` Java
String s = "Duplicate";
```
## 如何重用对象

通过使用静态工厂方法（详见第 1 条）和构造器，可以避免创建不需要的对象。例如，工厂方法 Boolean.valueOf(String) 比构造方法 Boolean(String) 更可取，后者在 Java 9 中被弃用。构造方法每次调用时都必须创建一个新对象，而工厂方法永远不需要这样做，在实践中也不需要。**除了重用不可变对象，如果知道它们不会被修改，还可以重用可变对象。**

　一些对象的创建比其他对象的创建要昂贵得多，可以将其缓存起来以便重复使用。

## 重用对象的例子
1. 将对象缓存起来

为了提高性能，作为类初始化的一部分，将正则表达式显式编译为一个 Pattern 实例（不可变），缓存它，并在 isRomanNumeral 方法的每个调用中重复使用相同的实例


```Java
import java.util.regex.Pattern;

// Reusing expensive object for improved performance (Pages 22 and 23)
public class RomanNumerals {
    // Performance can be greatly improved! (Page 22)
    static boolean isRomanNumeralSlow(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }

    // Reusing expensive object for improved performance (Page 23)
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }

    public static void main(String[] args) {
        int numSets = Integer.parseInt(args[0]);
        int numReps = Integer.parseInt(args[1]);
        boolean b = false;

        for (int i = 0; i < numSets; i++) {
            long start = System.nanoTime();
            for (int j = 0; j < numReps; j++) {
                b ^= isRomanNumeralSlow("MCMLXXVI");  // Change Slow to Fast to see performance difference
            }
            long end = System.nanoTime();
            System.out.println(((end - start) / (1_000. * numReps)) + " μs.");
        }

        // Prevents VM from optimizing away everything.
        if (!b)
            System.out.println();
    }
}
```

在经常调用时，改进版本的性能会显著提升，这样为不可见的 Pattern 实例创建静态 final 修饰的属性，并允许给它一个名字，这个名字比正则表达式本身更具可读性。

如果该改进方法从未被调用，则 `ROMAN` 属性则没必要初始化,因此，如果有必要的话，甚至还可以通过延迟初始化（lazily initializing）属性来排除初始化[条目83],但一般不建议这样做,延迟初始化常常会导致实现复杂化，而性能没有可衡量的改进[条目67]

2 避免使用自动装箱功能
``` Java
// Hideously slow! Can you spot the object creation?
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum;
}
```

**优先使用基本类型而不是装箱的基本类型，也要注意无意识的自动装箱。**

## 创建额外对象的好处

这个条目不应该被误解为暗示对象创建是昂贵的，应该避免创建对象。 相反，使用构造方法创建和回收小的对象是非常廉价，构造方法只会做很少的显示工作，尤其是在现代 JVM 实现上。 **创建额外的对象以增强程序的清晰度，简单性或功能性** 通常是件好事。

相反，**除非池中的对象非常重量级，否则通过维护自己的对象池来避免对象创建是一个坏主意**。对象池的典型例子就是数据库连接。建立连接的成本非常高，因此重用这些对象是有意义的。但是，一般来说，**维护自己的对象池会使代码混乱，增加内存占用，并损害性能**。现代 JVM 实现具有高度优化的垃圾收集器，它们在轻量级对象上轻松胜过此类对象池。

## 针对防御性复制的要求
条目 50 的防御性复制（defensive copying）指出：「不要重复使用现有的对象，当你应该创建一个新的对象时。」

> 重用防御性复制所要求的对象所付出的代价，要远远大于不必要地创建重复的对象。 未能在需要的情况下防御性复制会导致潜在的错误和安全漏洞；而不必要地创建对象只会影响程序的风格和性能。
