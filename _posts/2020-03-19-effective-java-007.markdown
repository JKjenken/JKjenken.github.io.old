---
layout:     post
title:      "07 - 消除过期的对象引用 "
subtitle:   "Effective-java-3rd"
date:       2020-03-19 18:00:00
author:     "linjiankai"
header-img: "img/post-bg-effective-java-3rd.png"
catalog: true
tags:
    - 学习
    - effective-java
---
Java自带垃圾收集机制，比起使用手动内存管理的语言（如 C 或 C++），程序员的工作会轻松的多，但是这不意味着可以完全不需要考虑内存管理。

## 无意的对象保留

``` Java
// Can you spot the "memory leak"?
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * Ensure space for at least one more element, roughly
     * doubling the capacity each time the array needs to grow.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
该程序有一个内存泄漏的可能，由于垃圾回收器的活动的增加，或内存占用的增加，静默地表现为性能下降。极端情况下，可能导致磁盘分页（disk paging），甚至导致内存溢出（OutOfMemoryError）的失败。

内存泄漏的问题就在于，当一个栈增长后收缩，那么从栈弹出的对象不会被垃圾收集，即使使用栈的程序不再引用这些对象。这就来自于栈维护对这些对象的过期引用（永远不会解除的引用）。

垃圾收集语言中的内存泄漏（无意的对象保留 unintentional object retentions）是隐蔽的。如果无意中保留了对象引用，那么不仅这个对象排除在垃圾回收之外，而且该对象引用的任何对象也是如此。 即使只有少数对象引用被无意地保留下来，也可以阻止垃圾回收机制对许多对象的回收，这对性能产生很大的影响。

##手动取消过期引用

一旦对象引用过期，将它们设置为 null。 在我们的 Stack 类的情景下，只要从栈中弹出，元素的引用就设置为过期。 pop 方法的修正版本如下所示：
```Java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

取消过期引用的另一个好处是，如果它们随后被错误地引用，程序立即抛出 NullPointerException 异常，而不是悄悄地继续做错误的事情。尽可能快地发现程序中的错误是有好处的。

## 不要过分反应

在程序结束后立即清除所有的对象引用是不可取的，会显得代码很混乱。清空对象引用应该是额外的而不是一种规范。消除过期引用最好的方法是 **不要让包含引用的变量超出它们应该存在的范围** 。也就是说在定义每个变量时尽量在其最近的作用域范围内定义[条目 57最小化局部变量的作用域]。

上面列举的Stack类之所以存在内存泄漏的问题，原因就是它管理自己的内存，elements数组的元素组成了一个存储池(storage pool),非活动的元素是空闲的，但是对于垃圾收集器来说，其内部的所有对象引用都是同样有效的。需要由程序员手动指定是否要清空这部分元素的引用，当其变成非活动部分时。

**当一个类自己管理内存时，我们应该警惕内存泄漏问题**，每当一个元素被释放时，元素中包含的任何对象引用都应该被清除。

## 缓存带来的内存泄漏

一旦将对象引用放入缓存中，很容易忘记它的存在，并且在它变得无关紧要之后，仍然保留在缓存中。对于这个问题有几种解决方案。如果你正好想自己实现一个缓存：只要在缓存之外存在对某个项（entry）的键（key）引用，那么这项就是明确有关联的，就可以用 WeakHashMap 来表示缓存；这些项在过期之后自动删除。记住，只有当缓存中某个项的生命周期是由外部引用到键（key）而不是值（value）决定时，WeakHashMap 才有用。
> 关于 WeakHashMap，简单来说，WeakHashMap会在系统内存范围内，保存所有键值数据，一旦内存不够，在GC时，没有被引用的键很快会被清除掉，从而避免系统内存溢出。

当缓存项的生命周期不太明确时，可以让其偶尔清理一次废弃的项，可以通过类似ScheduledThreadPoolExecutor这样的线程（定时执行任务）

```Java
 ScheduledExecutorService CleanExpired =  new ScheduledThreadPoolExecutor(1).
 swapExpiredPool.scheduleWithFixedDelay(() -> {
 					LOGGER.info("执行任务");
 				}, 1, 1, TimeUnit.DAYS);
```

或者在将新的项添加到缓存时顺便清理:可以通过LinkedHashMap的removeEldestEntry方法实现。更复杂的缓存可以直接使用 `java.lang.ref` 包。

## 监听器和其他回调带来的内存泄漏

如果你实现了一个 API，其客户端注册回调，但是没有显式地撤销注册回调，除非采取一些操作，否则它们将会累积。可以通过将其保存在WeakHashMap的键（key）中，只存储弱引用（weak references），从而确保回调被垃圾收集。

因为内存泄漏通常不会表现为明显的故障，所以它们可能会在系统中保持多年。 通常仅在 **仔细的代码检查或借助堆分析器（heap profiler）的调试工具才会被发现**。 因此，学习如何预见这些问题，并防止这些问题发生，是非常值得的。
