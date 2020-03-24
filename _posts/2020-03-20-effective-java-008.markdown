---
layout:     post
title:      "08 - 避免使用Finalizer和Cleaner机制 "
subtitle:   "Effective-java-3rd"
date:       2020-03-20 11:00:00
author:     "linjiankai"
header-img: "img/post-bg-effective-java-3rd.png"
catalog: true
tags:
    - 学习
    - effective-java
---
Java自带垃圾收集机制，比起使用手动内存管理的语言（如 C 或 C++），程序员的工作会轻松的多，但是这不意味着可以完全不需要考虑内存管理。

## 什么是Finalizer

Finalizer顾名思义是一个终结器，在Object中，有一个叫finalize的方法，该方法的实现是空的，但是其可以用来终结进程，当jvm检测到一个类有一个finalize()方法时，Finalizer 机制线程的运行优先级低于其他应用程序线程，所以对象被回收的速度低于进入队列的速度。

##Finalizer的缺点

Finalizer 机制的一个缺点是不能保证他们能够及时执行，在一个对象变得无法访问时，到 Finalizer 和 Cleaner 机制开始运行时，这期间的时间是任意长的。 这意味着你永远不应该 Finalizer 和 Cleaner 机制做任何时间敏感（time-critical）的事情。 例如，依赖于 Finalizer 和 Cleaner 机制来关闭文件是严重的错误，因为打开的文件描述符是有限的资源。 如果由于系统迟迟没有运行 Finalizer 和 Cleaner 机制而导致许多文件被打开，程序可能会失败，因为它不能再打开文件了。

另一个问题是在执行 Finalizer 机制过程中，未捕获的异常会被忽略，并且该对象的 Finalizer 机制也会终止 [JLS, 12.6]。未捕获的异常会使其他对象陷入一种损坏的状态（corrupt state）。如果另一个线程试图使用这样一个损坏的对象，可能会导致任意不确定的行为。

不要相信 System.gc 和 System.runFinalization 方法。 他们可能会增加 Finalizer 和 Cleaner 机制被执行的几率，但不能保证一定会执行。

使用 finalizer 和 cleaner 机制会导致严重的性能损失。

finalizer 机制有一个严重的安全问题：它们会打开你的类来进行 finalizer 机制攻击。**如果一个异常是从构造方法或它的序列化中抛出的——readObject 和 readResolve 方法 （第 12 章）——恶意子类的 finalizer 机制可以运行在本应该「中途夭折（died on the vine）」的部分构造对象上。**
