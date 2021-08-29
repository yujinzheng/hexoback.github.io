---
title: 静态工厂方法
tags:
  - 创建和销毁对象
  - Effective Java
categories:
  - 代码沉思录
date: 2021-08-28 21:50:10
---


## 什么是静态工厂方法

在Java中，获取一个类实例最常用的方法就是使用`new`关键字，通过构造函数来实现对象的创建，比如这样：

```java
List<String> myList = new ArrayList<>();

JSONObject jsonObject = new JSONObject();
```

但是在实际开发中，我们经常能够见到另一种获取类实例的方法：

```java
String testStr = String.valueOf(123);

Integer integer = Integer.getInteger("123");

Calendar calendar = Calendar.getInstance();
```

如上面这种不使用`new`，而是用一个静态方法对外提供自身实例的方法，就是`静态工厂方法(Static Factory Method)`。

<!-- more --> 

## 为什么使用静态工厂方法

在《Effective Java》这本书中，第一条就是“考虑使用静态工厂方法代替构造器”。在书中，作者给出了使用静态工厂方法的几个优势：

**1. 静态工厂方法与参数构造器不同的是，它们有名称**

由于语言的特性，Java的构造函数跟类名是一样的，在有多个重载的构造函数的情况下，构造函数很难准确地描述返回值或创建的对象的性质，尤其是当参数类型、参数数目又比较类似时，更容易出错。

如下面的代码：

```java
Date date0 = new Date();
Date date1 = new Date(1L);
Date date2 = new Date("1");
Date date3 = new Date(1,2,3);
Date date4 = new Date(1,2,3,4,5);
Date date5 = new Date(1,2,3,4,5,6);
```

如果开发者对Date类不够熟悉，在选用构造函数的时候可能还需要查看文档或源码才能明白每个参数的含义。而如果使用静态工厂方法的话，就可以给方法起更多有意义的名字，比如`valueOf`、`getDataByString`、`newInstance`、`getInstance`等，有助于代码的编写和阅读。

**2. 静态工厂方法不必在每次调用时都创建一个新的对象**

有的时候，外部调用者只需要拿到一个实例，二不关心拿到的是否是一个新的实例；又或者我们只想对外提供一个单例的时候，静态工厂方法能够很好地满足我们的诉求。

**3. 静态工厂方法可以返回原返回类型的任何子类型对象**

构造函数只能返回确切的自身类型，而静态工厂方法则能够更加灵活，可以根据需要返回任何它的子类型实例。

```java
class Clothes {
    public static Clothes getInstanceByType(String type) {
        switch (type) {
            case "shirt":
                return new Shirt();
            case "jacket":
                return new Jacket();
            case "skirt":
                return new Skirt();
            default:
                System.out.println("Can not find your clothes");
        }
    }
}

class Shirt extends Clothes {
}

class Jacket extends Clothes {
}

class Skirt extends Clothes {
}
```

**4. 在创建参数化类型实例的时候，能够是代码变得更加简洁**

这一条主要是针对参数化类型的声明而说的，比如在Java7之前，构造参数化类型需要重复书写两次泛型参数：

```java
Map<String, List<String>> m = new HashMap<String, List<String>>();
```

不过从Java7开始，这种方式已经被优化了：对一个一直类型的变量进行赋值时，由于类型推导（type inference）的原因，编译器可以帮你找到类型参数：

```java
Map<String, List<String>> m = new HashMap<>();
```

除了《Effective Java》中给出的几条理由，我们还能够找到静态工厂方法带来的更多好处：

**5. 可以减少对外暴露的属性**

例如在初始化实例的时候，我们希望直接给实例配置某些固定属性，这个时候，就可以通过静态工厂方法来实现了：

```java
class Phone {
    public static final String  BRANDS_APPLE = "Apple";

    public static final String  BRANDS_HUAWEI = "Huawei";

    public static final String  BRANDS_SAMSUNG = "Samsung";

    public static final String  BRANDS_XIAOMI = "Xiaomi";

    private String brands;

    private Phone (String brands) {
        this.brands = brands;
    }

    public static Phone newApple() {
        return new Phone(BRANDS_APPLE);
    }

    public static Phone newHuawei() {
        return new Phone(BRANDS_HUAWEI);
    }

    public static Phone newSamsung() {
        return new Phone(BRANDS_SAMSUNG);
    }

    public static Phone newXiaomi() {
        return new Phone(BRANDS_XIAOMI);
    }
}
```

通过将构造函数声明为`private`，防止被外部调用，调用方在创建手机时，只能通过静态工厂方法来创建，这样，调用方无须制定brands值，能够减少对外暴露的属性。

**6. 使用静态工厂方法可以将一系列行为封装起来，有利于代码管理**

比如在编写测试数据时，可以用一个专用的静态工厂方法来获取测试实例，一旦需要正式上线，就可以在类里面修改或删除这个方法，那么所有的调用就都能同步到变化，减少测试代码上线的可能性。

```java
class Person {
    private String name;
    private int age;
    private int height;

    public static Person newTestInstance() {
        Person tester = new Person();
        tester.setName("王大全");
        tester.setAge(10000);
        tester.getHeight(5000);
        return tester;
    }
}

// 其他地方调用测试实例
Person tester = Person.newTestInstance();

//其他针对tester的操作
```

当然，静态工厂方法也存在一些缺点：

**1. 类如果不含有共有的或者受保护的构造器，就不能被子类化**

实际上我觉得这个并不算太大的问题，因为只要注意对外提供合适的构造器就可以了，例如对于`String`类，我们既可以使用`valueOf`方法，也可以使用`new`来获取实例。

同时，不能子类化可能也会让我们思考，是否有比继承更好的方法来实现我们的功能，比如复合（composition），说不定我们反而能因祸得福。

**2. 静态工厂方法与其他静态方法实际上没有任何区别**

由于静态工厂方法本质上就是静态方法，因此在API文档中不会被明确标识出来，面对这种情况，我们只能尽量使用约定成俗或标准的命名习惯来弥补这一个劣势。

下面是静态工厂方法的一些惯用名称：

-  valueOf
- of
- getInstance
- newInstance
- getType
- newType

总而言之，静态工厂方法和公有构造器都各有用处，我们需要理解它们各自的优势，根据实际情况选用——当然，静态工厂方法往往更加合适，因此，切忌第一反应就是提供公有的构造器，而是优先考虑使用静态工厂方法的可能性。




















