---
layout:     post
title:      "02 - 当构造方法参数过多时使用 builder 模式 "
subtitle:   "Effective-java-3rd"
date:       2020-03-10 09:40:00
author:     "linjiankai"
header-img: "img/post-bg-effective-java-3rd.png"
catalog: true
tags:
    - 学习
    - effective-java
---
## 类的构建方式
当类包含更多的可选参数时，有三种可选方案。
1. 可伸缩构造方法模式
可伸缩构造方法模式是有效的，但是当有很多参数时，很难编写客户端代码，而且很难读懂它。
2. JavaBeans模式
调用一个无参的构造方法来创建对象，然后调用 setter 方法来设置每个必需的参数和可选参数。
```Java
// JavaBeans Pattern - allows inconsistency, mandates mutability
public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize  = -1; // Required; no default value
    private int servings     = -1; // Required; no default value
    private int calories     = 0;
    private int fat          = 0;
    private int sodium       = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }

    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)    { servings = val; }
    public void setCalories(int val)    { calories = val; }
    public void setFat(int val)         { fat = val; }
    public void setSodium(int val)      { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
}
```
JavaBeans 模式本身有严重的缺陷。由于构造方法被分割成了多次调用，所以在构造过程中JavaBean 可能处于不一致的状态。该类没有通过检查构造参数的参数有效性来强制一致性的选项。在不一致的状态下尝试使用对象可能会导致一些错误，这些错误与平常代码的 BUG 很是不同，因此很难调试。
另一个缺点是JavaBeans模式排除了让类不可变的可能性，并且需要程序员增加工作以确保线程安全。
3. Builder模式
客户端不直接构造所需的对象，而是调用一个包含所有必需参数的构造方法 (或静态工厂) 得到一个 builder 对象。
```Java
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val) {
            calories = val;      
            return this;
        }

        public Builder fat(int val) {
           fat = val;           
           return this;
        }

        public Builder sodium(int val) {
           sodium = val;        
           return this;
        }

        public Builder carbohydrate(int val) {
           carbohydrate = val;  
           return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```
NutritionFacts 类是不可变的，所有的参数默认值都在一个地方。builder 的 setter 方法返回 builder本身，这样就可以进行链式调用，从而生成一个流畅的 API。
```Java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100).sodium(35).carbohydrate(27).build();
```
**实际应用项目时，不需要手动编写繁琐的builder样板代码，可使用lombok编写样板代码**

该builder模式除了基本用法之外还有更为高级的使用注意事项。
## 编写防御性拷贝
由于类的客户端在使用的过程中很可能会摧毁类的不变性，所以需要在类的构造方法中增加防御性的代码，在builder模式时，需要检查 builder 的构造方法和方法中的参数有效性，在build方法的调用创建对象的位置上编写代码，检查多个参数的不变性，为了确保不被攻击，需要对对象属性进行检查，检查失败时抛出IllegalArgumentException 异常（优先使用标准的异常），在异常消息捕获处指出哪些参数无效。
## 在类层次结构中使用平行层次的builder
每个builder嵌套在相应的类中。 抽象类有抽象的 builder；具体的类有具体的 builder。
代表各种比萨饼的根层次结构的抽象类：
```Java
// Builder pattern for class hierarchies
import java.util.EnumSet;
import java.util.Objects;
import java.util.Set;

public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // Subclasses must override this method to return "this"
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // See Item 50
    }
}
```
Pizza.Builder 是一个带有递归类型参数（ recursive type parameter）（030 优先使用泛型代码）的泛型类型。该嵌套类与抽象的 self 方法一起，允许方法链在子类中正常工作，而不需要强制转换。

>  编写具体的子类实现该抽象类

```Java
import java.util.Objects;

public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}

public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // Default

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }

        @Override protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```

一个子类的方法被声明为返回在超类中声明的返回类型的子类型，称为协变返回类型（covariant return typing）。它允许客户端使用这些 builder，而不需要强制转换。

客户端代码的编写(假设枚举常量使用静态导入)

``` Java
NyPizza pizza = new NyPizza.Builder(SMALL)
        .addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder()
        .addTopping(HAM).sauceInside().build();
```
## build模式的优势
* builder 可以有多个可变参数，因为每个参数都是在它自己的方法中指定的。
* builder模式可以将传递给多个调用的参数聚合到单个属性中，如上面代码中超类的addTopping方法一样。
* 单个 builder 可以重复使用来构建多个对象。 builder的参数可以在构建方法的调用之间进行调整，以改变创建的对象。
* builder 可以在创建对象时自动填充一些属性，例如每次创建对象时增加的序列号。
* builder模式客户端代码比使用伸缩构造方法（telescoping constructors）更容易读写，并且builder模式比 JavaBeans 更安全。
## build模式的缺点
* 创建对象的 builder需要成本（影响很小）
* builder 模式比伸缩构造方法模式更冗长,如果当类演化到参数数量失控的时候再转到Builder模式，过时的构造方法或静态工厂就会难以修改。
