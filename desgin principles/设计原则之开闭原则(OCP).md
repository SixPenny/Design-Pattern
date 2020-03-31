## 简介

软件是一直在变化之中的。如何应对这些变化是开闭原则要解决的问题。开闭原则允许软件实体在不更改其代码的情况下变更其行为（变更包括改变和增加）。

## 定义

开闭原则(Open Close Principle) 是面向对象设计中重要的原则，它要求**软件实体对扩展开放，对修改封闭**，软件实体包含函数、类、模块甚至是可执行程序。

开闭原则可以基于两种实现：一种是基于接口，一种是基于抽象类。接口是实体之间交互的规约，只要实现是符合规约的，使用方就无需关心具体细节，这种情况下替换实现不影响软件实体功能。抽象类也是一种规约，只不过抽象类所代表的规约约束更强，因为抽象类可以定义流程，子类只需实现自身变化的部分即可。

## 插件与中间件

插件与中间件可以说必须遵守开闭原则。软件在发布后使用人员是无法修改源代码的，因此源代码设计人员需将能力开放给使用人员，就必须依赖开闭原则。

有关这方面，有人提出了 SPI 的概念，可参加以前写的[API 与 SPI](https://my.oschina.net/liufq/blog/3011988)

## 与设计模式之间的关系

设计原则是设计模式的指导，设计模式是设计原则的具象。比较虚的原则需要使用比较实的模式来说明。很多文章使用策略模式来讲解开闭原则，导致很多人认为开闭原则就是策略模式，实际上策略模式是开闭原则的一种实现而已。

## 实践

在订单付款完成后，系统需要向用户发送付款成功通知，向卖家发送订单通知，还需要向 BI 部分发送通知用于统计。

使用函数来完成功能：
```java
public class OrderServiceImpl {
    public void notify(Order order) {
        notifyBuyer(order);
        notifySeller(order);
        notifyBi(order);
    }
}
```

如果系统中有另一方对付款成功的通知感兴趣，那就需要修改 `notify` 函数来增加这一功能。按照开闭原则，应该尽量做到修改封闭。

改为使用事件方式来通知各方：
```java
public class OrderPaiedListener {
    public void orderPaied(Order order);
}

public class OrderServiceImpl {
    private List<OrderPaiedListener> paiedListeners = new LinkedList<>();
    public void notify(Order order) {
        for (OrderPaiedListener listener : paidListeners) {
            listener.orderPaied(order);
        }
    }    
}

```
通过将事件接收方抽象成一个观察者，系统在通知函数里就只需要遍历所有的观察者，将事件发送给它们就可以了，而不需要关系具体通知的处理逻辑，也不关心谁对通知感兴趣。

如果有新的接收方，只需要实现接口并将自己注册到容器中即可，订单付款通知的逻辑不需要变化。

## 优缺点

优点：
- 可以减少单元测试、功能测试的成本，增加质量保证。
- 关注点分离
- 可扩展性强
- 可维护性强
- 对合作友好，多人增加代码不会发生冲突

缺点：
- 类增多
- 容易衍生过度设计
- 代码散落各地，如果接口实现对流程有影响，定位问题比较困难（如Spring 的 `BeanFactoryPostProcessor`会根据返回的 bean 是否为 null 决定是否继续向后）
