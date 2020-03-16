---
layout:     post
title:      "03 - 使用私有构造方法或枚类实现 Singleton 属性 "
subtitle:   "Effective-java-3rd"
date:       2020-03-11 18:20:00
author:     "linjiankai"
header-img: "img/post-bg-effective-java-3rd.png"
catalog: true
tags:
    - 学习
    - effective-java
---
## 单例模式的含义

单例是一个仅实例化一次的类，通常用来表示无状态的对象，如一个函数或者一个唯一的系统组件。将类设计成单例模式对于客户端来说测试起来会变得困难。
原文的描述是：
> Making a class a singleton can make it difficult to test its clients, as it’s impossible to substitute a mock implementation for a singleton unless it implements an interface that serves as its type.

这意味着需要为单例类设置实现测试的接口，并且使用该接口将单例注入测试代码。

```Java
public class SomeSingleton implements SomeInterface {
    ...
}

@Before
public void resetSingleton() throws SecurityException, NoSuchFieldException, IllegalArgumentException, IllegalAccessException {
   Field instance = SomeSingleton.class.getDeclaredField("SomeInterface");
   instance.setAccessible(true);
   instance.set(null, null);
}
```
## 实现单例的方法

实现单例时均需要保持构造方法私有和导出公共静态成员以提供对唯一实例的访问。

1. 提供私有构造方法和公共静态final属性，只调用一次私有构造方法。

```Java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}

```

客户端可以使用 AccessibleObject.setAccessible 方法，以反射方式调用私有构造方法，如果要防御该攻击，可修改构造函数，使其在创建第二个实例的时候抛出异常。

2. 使用静态工厂方法作为公共成员

```Java
// Singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}
```
公共属性方法的主要优点是 API 明确表示该类是一个单例,这让我们可以很容易通过修改该方法从而修改类的是否单例化，或者改为泛型的单例工厂，或者可以通过方法引用去调用它。如果不需要上述优点，可优先考虑public-field（方法1）的方式实现。
## 使用注意事项
为了确保该单例是可序列化的，除了添加 `implements Serializable` 声明外，还必须声明所有的实例字段为`transient`（生命周期仅存于调用者的内存中而不会写到磁盘里持久化），并提供一个`readResolve` 方法，否则，每当序列化的实例被反序列化时，就会创建一个新的实例。
在实例类中新增`readResolve`方法：
```java
// readResolve method to preserve singleton property
private Object readResolve() {
     // Return the one true Elvis and let the garbage collector
     // take care of the Elvis impersonator.
    return INSTANCE;
}
```
## 更简便的方式创建单例模式
声明单一元素的枚举类
```Java
// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```
这种方式类似于公共属性方法，但更简洁，无偿地提供了序列化机制，并提供了防止多个实例化的坚固保证，即使是在复杂的序列化或反射攻击的情况下。单一元素枚举类通常是实现单例的最佳方式。

注意，如果单例必须继承 `Enum`以外的父类 (尽管可以声明一个 Enum 来实现接口)，那么就不能使用这种方法。
