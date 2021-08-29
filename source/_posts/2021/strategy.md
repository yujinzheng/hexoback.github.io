---
title: 策略模式
date: 2021-08-24 01:48:25
tags:
- 设计模式
- 对象行为模式
categories:
- 代码沉思录
---

## 设计模式的定义与特点

在现实生活中，我们经常遇到实现目标存在多种策略可供选择的场景，比如出行的时候可以步行、乘公交、乘地铁、自驾等，付款时可以选择现金支付、支付宝支付、微信支付等。

在软件开发中叶经常遇到类似的情况，某种方案的实现可以采用多种算法或策略，我们可以根据实际情况采取不同的算法或策略来完成功能。如果我们使用条件语句来选择策略或算法，那么每当策略变更时，我们可能都会面临原代码的修改，不利于代码的维护，也违背了“软件对扩展开放，对修改关闭”的设计原则，因此，我们引入了**策略模式**。

<!-- more --> 

**定义**：策略模式定义了一系列算法，并将每个算法封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

策略模式的主要优点如下：

1. 策略模式提供了一系列可供重用的算法族，可以通过继承将算法族的公共代码转移到父类中，减少重复代码
2. 策略模式可以提供相同行为的不同实现，客户可以根据实际请款选择不同的策略
3. 策略模式满足了“软件对扩展开放，对修改关闭”的设计原则，可以在不修改原代码的情况下，灵活新增策略

策略模式的主要缺点如下：

1. 客户必须知道所有策略算法的区别，以便适时选择恰当的算法
2. 策略模式造成了很多策略类，增加维护难度

## 策略模式的结构与实现

根据策略模式的定义，策略模式由两部分组成：持有策略引用的环境类以及一系列的算法族，考虑到“多用组合，少用继承”的设计原则与“算法之间可以互相替换”的要求，这些算法族实现同一个接口，并且复写同一个调用方法。策略模式的重心不是如何实现算法，而是如何组织算法，从而让程序结构更加灵活，具有良好的维护性和扩展性。

### 模式的结构

策略模式的主要角色如下：

1. 算法接口：算法族中的算法都需要实现这个接口，并且覆写同一个调用方法
2. 具体算法类：实现了算法接口，提供具体的算法实现
3. 环境类：持有一个算法接口的引用，客户可以使用环境类来调用不同的策略

其结构图如图1所示：

<div>
    <center>
    <img src="../images/strategy01.png"
    alt="策略模式结构图">
    图1 策略模式结构图
    </center>
</div>

### 策略模式的实现

策略模式实现的代码如下：
```java
public class StrategyPattern {
    public static void main(String[] args) {
        Context context = new Context();
        Strategy concreteStrategyA = new ConcreteStrategyA();
        Strategy concreteStrategyB = new ConcreteStrategyB();
        context.setStrategy(concreteStrategyA);
        context.strategyMethod();
        System.out.println("=====================");
        context.setStrategy(concreteStrategyB);
        context.strategyMethod();
    }
}

public interface Strategy {
    public void strategyMethod();
}

public class ConcreteStrategyA implements Strategy{
    public void strategyMethod() {
        System.out.println("执行策略A");
    }
}

public class ConcreteStrategyB implements Strategy{
    public void strategyMethod() {
        System.out.println("执行策略B");
    }
}

public class Context {
    Strategy strategy;

    public void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }

    public Strategy getStrategy() {
        return strategy;
    }

    public void strategyMethod() {
        strategy.strategyMethod();
    }
}
```

程序执行的结果如下：
```text
执行策略A
=====================
执行策略B
```

### 策略模式的实例

**丁一的奇妙冒险**

背景：丁一是一个平平无奇的程序员，有一天他在加班时，鼠标神奇地被雷劈了，从此他就拥有了使用鼠标穿越的能力，当他点击鼠标左键时，他就会魂穿，当他点击鼠标右键时，他就会身穿，当他点击滚轮时，他就半身穿（比如穿越成半人马），当然，不排除还有更多的穿越模式，只是丁一暂时还没发现而已，作为平平无奇的程序猿，丁一想到了用策略模式来描述自己的穿越能力。

首先，定义一个穿越策略的接口（TraversalStrategy），里面包含了一个穿越方案的抽象方法startTraversal()；然后，定义一系列穿越策略，比如身穿（BodyTraversal）类、魂穿（SoulTraversal）类、半身穿（HalfBodyTraversal）类；最后，定义环境上下文，在这里就是丁一的鼠标（Mouse），丁一通过鼠标来触发穿越策略。

程序代码如下：

```java
public class Main {
    public static void main(String[] args) {
        Mouse mouse = new Mouse();
        System.out.println("我是丁一，我今天准备使用鼠标进行穿越");
        mouse.setTraversalStrategy(new BodyTraversal());
        mouse.startTraversal();
        mouse.setTraversalStrategy(new SoulTraversal());
        mouse.startTraversal();
        mouse.setTraversalStrategy(new HalfBodyTraversal());
        mouse.startTraversal();
    }
}

public interface TraversalStrategy {
    public void startTraversal();
}

public class BodyTraversal implements TraversalStrategy{
    public void startTraversal() {
        System.out.println("触发了身穿，整个人都穿越啦");
    }
}

public class HalfBodyTraversal implements TraversalStrategy{
    public void startTraversal() {
        System.out.println("触发了半身穿，只有一半的身体穿越啦");
    }
}

public class SoulTraversal implements TraversalStrategy{
    public void startTraversal() {
        System.out.println("触发了魂穿，只有灵魂穿越啦");
    }
}

public class Mouse {
    TraversalStrategy traversalStrategy;

    public void startTraversal() {
        traversalStrategy.startTraversal();
    }

    public void setTraversalStrategy(TraversalStrategy traversalStrategy) {
        this.traversalStrategy = traversalStrategy;
    }

    public TraversalStrategy getTraversalStrategy() {
        return traversalStrategy;
    }
}

```

程序输出结果如下：

```text
我是丁一，我今天准备使用鼠标进行穿越
触发了身穿，整个人都穿越啦
触发了魂穿，只有灵魂穿越啦
触发了半身穿，只有一半的身体穿越啦
```

从上面的例子中我们可以看到，如果丁一发现了新的穿越方式，那么就只需要新增穿越策略，而无需对他的鼠标进行修改，使得代码更加灵活。然而，如果丁一脑洞过大，发现了很多的穿越模式，那么不仅每种穿越模式需要一个策略，他本人也必须了解所有的穿越模式，这样会导致维护变得困难。

### 策略模式的应用场景

通常在以下几种情况中使用策略模式较多：

1. 一个系统需要动态地在几种算法中选择一种时
2. 系统中算法彼此完全独立，且要求对客户隐藏具体算法的实现希捷
3. 多个类只区别在表现行为不同