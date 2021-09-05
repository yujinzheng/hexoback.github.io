---
title: 当构造方法参数过多时使用Builder模式
date: 2021-09-05 19:49:01
tags:
  - 创建和销毁对象
  - Effective Java
categories:
  - 代码沉思录
---

静态工厂方法和构造方法都有一个限制：它们不能很好地扩展到多个可选参数的情景。考虑如下场景：

> 国家针对食品的营养成分表的规定中，除了强制要求的能量、蛋白质、脂肪和碳水化合物外，膳食纤维、胆固醇、糖、维生素和矿物质等都没有明确的规定。针对这些可选属性，就算产品想要标注，也一般只会选择部分非零值，而不会全部选择。

针对上面的场景，我们应该编写什么样的构造方法或静态工厂方法？最传统的方法，我们可使用可伸缩的构造方法模式，即首先提供一个只有必需参数的构造函数，接着提供增加了一个可选参数的构造函数，然后提供增加了两个可选参数的构造函数...，最终在构造函数中包含所有必需和可选参数。下面我们选择四个可选属性来看看实践中的代码是什么样的：

<!-- more --> 

```java
public class NutritionFacts {
    /**
     * 必需参数
     */
    private final int servingSize;
    private final int servings;

    /**
     * 可选参数
     */
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

大家可以想象，随着参数数量的增加，我们构造函数将会变得臃肿不堪，最后失去控制。简而言之，**可伸缩构造方法模式是有效的，但是当有很多参数时，很难编写客户端代码，而且很难读懂它**。比如上面的`NutritionFacts`类，我们在使用构造函数的时候，如果将参数的位置写反了，编译器并不会报错，但是程序在运行时会出现错误的行为。

当在构造函数中遇到很多可选参数时，另一种选择是`JavaBeans`模式，在这种模式中，调用一个无参的构造方法来创建对象，然后调用`setter`方法来设置每个必需的参数和可选参数：

```java
public class NutritionFacts {
    /**
     * 必需参数
     */
    private final int servingSize;
    private final int servings;

    /**
     * 可选参数
     */
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts() { }

    // Setters
    public void setServingSize(int val) { servingSize = val; }
    public void setServings(int val) { servings = val; }
    public void setCalories(int val) { calories = val; }
    public void setFat(int val) { fat = val; }
    public void setSodium(int val) { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
```

这种模式没有伸缩构造方法模式的缺点，虽然有点冗长，但是创建实例很容易，并且易于阅读所生成的代码：

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

不幸的是，`JavaBeans`模式本身有严重的缺陷。由于构造方法被分成了多次调用，所以在构造过程中`JavaBeans`可能处于不一致的状态。该类没有通过检查构造参数的有效性来强制一致性的选择。在不一致的状态下尝试使用对象可能会导致一些错误，这些错误与平常代码的Bug很不同，因此很难调试。一个相关的缺点是，`JavaBeans`模式排除了让类不可变的可能性，并且需要保证线程安全。

通过在对象构建完成时手动「冻结」对象，并且不允许它在解冻之前使用，可以减少这些缺点，但是实践中很难并很少使用，并且在试运行时可能会导致错误，因为编译器无法确保程序猿会在使用对象之前调用`freeze`方法。

接下来，就是本条中提到的Builder模式了，它结合了可伸缩构造方法模式的安全性和`JavaBeans`模式的可读性。这种模式通过调用一个包含所有必需参数的构造方法（或静态工厂方法）获得一个builder对象；然后，客户端调用builder对象的与`setter`相似的方法来设置可选参数；最后，客户端调用builder对象的一个无参的`build`方法来生成对象，该对象通常是不可变的。Builder通常是它所构建的类的一个静态成员类。下面是我们用Builder模式对营养成分表类的改造实例：

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        /**
        * 必需参数
        */
        private final int servingSize;
        private final int servings;

        /**
        * 可选参数
        */
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
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
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

`NutritionFacts`类是不可变的，所有的参数默认值都在一个地方。builder的`setter`方法返回builder本身，这样就可以进行链式调用，从而生成一个流畅的API。西面是客户端代码的实例：

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();
```

这个客户端代码容易编写且易于阅读。在Python和Scala中，都可以找到采用Builder模式的的例子。

实例中忽略了有效性检查，builder需要检查构造方法和方法中参数的有效性。在`build`方法调用的构造方法中检查包含多个参数的不变性。为了确保这些不变性不受攻击，在从builder赋值参数后，对对象进行检查，如果检查失败，则抛出`IllegalArgumentException`异常，其详细消息指示哪些参数无效。

Builder模式非常适合类层次结构，使用平行层次的builder，每个builder嵌套在相应的类中。抽象类有抽象的builder；具体类有具体的builder。例如，考虑代表各种披萨饼的根层次结构的抽象类：

```java
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
        
        // 子类必须复写此方法以return this
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
```

请注意，`Pizza.Builder`是一个带有递归类型参数的泛型类型，它与抽象的`self`方法一起，允许方法链在子类中正常工作，而不需要强制转换。

下面是两个具体的`Pizza`子类，其中一个是标准的纽约风格披萨，有一个所需的尺寸参数，另一个是半圆形烤乳酪馅饼，允许指定酱汁在里面还是在外面：

```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
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
        private boolean sauceInside = false;

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```

每个子类builder方法中的`build`方法被声明为返回正确的子类：`NyPizza.Builder`的`build`方法返回`NyPizza`，而`Calzone.Builder`中的`build`方法返回`Calzone`。这种一个子类的方法被声明为在超类中声明的返回类型的子类型，被称为**协变返回类型**，它允许客户端使用这些builder而不需要强制转换。

> 协变返回类型：子类重写基类方法时，返回的类型可以是基类方法返回类型的子类。协变返回类型允许返回更为具体的类型。此特性于Java 5.0添加。

这些分层builder的客户端代码基本上与简单的`NutritionFacts`builder的代码相同，下面显示披萨客户端代码枚举常量的静态导入：

```java
NyPizza pizza = new NyPizza.Builder(SMALL).addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder().addTopping(HAM).sauceInside().build();
```

builder方法相对于构造方法的一个小优势是，builder可以由多个可变参数，因为每个参数都是在它自己的方法中指定的。或者说，builder可以将传递给多个调用的参数聚合到单个属性中，如前面的`addTopping`方法所演示的那样。

Builder模式非常灵活，单个builder可以重复使用来构建多个对象。builder的参数可以在构建方法的调用之间进行调整以改变创建的对象。builder可以在创建对象时自动填充一些属性，如每次创建对象时增加序列号等。

Builder模式也有一些缺点：

1. 为了创建对象，首先必须创建它的builder。在看重性能的场景下，这种成本可能是一个问题。
2. Builder模式比伸缩构造方法更冗长，因此只有在有足够的参数时才值得使用它，一般建议四个或更多参数的时候才考虑使用Builder模式。

**请注意：**如果一开始使用的是静态工厂方法或构造方法，当类演化到参数数量失控的时候再转到Builder模式时，静态工厂方法或构造方法将面临尴尬的处境。因此，通常最好从一开始就创建一个builder。

总之，当设计类的构造方法或静态工厂方法的参数超过几个时，特别是如果许多参数是可选的活相同的类型时，Builder模式是一个不错的选择。Builder模式的客户端代码通常比使用伸缩构造方法更容易读写，并且Builder模式比`JavaBeans`更安全。