---
title: 使用私有构造方法或枚举类实现Singleton属性
tags:
  - 创建和销毁对象
  - Effective Java
categories:
  - 代码沉思录
date: 2021-09-21 23:18:00
---

单例是一个仅实例化一次的类。单例对象通常表示无状态对象，如函数或一个本质上唯一的系统组件。让一个类成为单例会使测试它的客户端变得困难，因为除非实现一个作为它（该单例）类型的接口，否则不可能用一个模拟实现替代单例。

有两种常见的方法来实现单例。二者都基于保持构造方法私有和导出公共静态成员以提供对唯一实例的访问。在第一种方法中，成员是`final`修饰的属性：

```java
// 使用public final修饰属性的单例
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}
```

<!-- more --> 

私有构造方法只调用一次，用于初始化公共静态属性`final Elvis INSTANCE`。缺少一个公共的或受保护的构造方法，保证了全局的唯一性：一旦Elvis类被初始化，一个Elvis的实例就会存在，客户端所做的任何事情都不能改变这一点，但需要注意的是：特权客户端可以使用`AccessibleObject.setAccessible`方法，以反射的方式调用私有构造方法。如果需要防御此攻击，请修改构造函数，使其在请求创建第二个实例时抛出异常。

在第一种实现单例的方法中，公共成员是一个静态的工厂方法：

```java
// 使用静态工厂方法的单例
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }
    public void leaveTheBuilding() { ... }
}
```

所有对`Elvis.getInstance`的调用都返回相同的对象引用，并且不会创建其他的Elvs实例。

公共属性方法的主要优点是API明确表示该类是一个单例：公共静态属性是final的，所以它总是包含相同的对象引用。第二个好处则是它更加简单。

静态工厂方法的优势之一在于，它提供了灵活性：在不改变其API的情况下，我们可以改变该类是否为单例的想法。工厂方法返回该类的唯一实例，但是，它很容易被修改，比如，改为每个调用该方法的线程返回一个唯一的实例。第二个优势是，如果你的应用程序需要，可以编写一个泛型单例工厂（generic singleton factory）。使用静态工厂方法的最后一个优势是，可以通过方法引用（method reference）作为提供者，例如`Elvis::instance`等同于`Supplier<Elvis>`。除非满足以上任意一种优势，否则还是有限考虑公有域方法。

为了将上述方法中实现的单例类编程可序列化的，仅仅将`implements Serializable`添加到声明中是不够的。为了保证单例模式不被破坏，必须声明所有的实例字段为`transient`，并提供一个`readResolve`方法。否则，每当序列化的是咧被反序列化时，就会创建一个新的实例。为了防止这种情况发生，可以将如下的`readResolve`方法添加到类中：

```java
// readResolve方法用于保证单例属性
private Object readResolve() {
    // 返回一个真正的Elvis类并让GC处理不必要的Elvis
    return INSTANCE;
}
```

实现一个单例的第三种方法是声明单一元素的枚举类

```java
// 枚举单例 —— 首选方法
public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() { ... }
}
```

这种方式类似于公共属性方法，但是更加简洁，并且无偿提供了序列化机制，即使是在复杂的序列化或反射攻击的情况下，它也能为防止多个实例化提供坚强的保证。这种使用方法可能有点不自然，但是**单一元素枚举类型是实现单例的最佳方式**。注意，如果单例必须继承`Enum`以外的父类（尽管还能声明一个`Enum`来实现接口），那么就不能使用这种方法。