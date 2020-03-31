## 简介

里氏替换原则是在做继承设计时需要遵循的原则，不遵循了 LSP 的继承类会带来意想不到的问题。

## 定义

里氏替换原则(Liskov Substitution Principle) 是由 Barbara Liskov  在 1987 年提出来的，Liskov 是她的姓，国内翻译成 里氏。

原则声明：如果类型 S 是类型 T 的子类型，那么 T 类型的对象可以替换成 S 类型的对象，而不会影响程序的行为。

LSP 对语言增加了新的签名约束(协变与逆变可以看这篇文章[Java中的逆变与协变](https://www.cnblogs.com/en-heng/p/5041124.html))：
- Contravariance of method arguments in the subtype.
- Covariance of return types in the subtype.
- No new exceptions should be thrown by methods of the subtype, except where those exceptions are themselves subtypes of exceptions thrown by the methods of the supertype.

从契约角度来看，里氏替换原则有4层含义：
1. 方法的前置条件要求不能更严格（可以更宽松）
2. 方法的后置条件不能更宽松（可以更严格）
3. 子类要保持父类约定的不变性
4. 历史约束。类属性只能通过方法来修改，由于子类会引入父类中不存在的方法，方法的引入可能会导致原来在父类中不可修改的属性在子类中可以修改了，历史约束禁止这种行为。

## 思考

继承描述的是 `is-a`  关系，开闭原则要求我们使用继承增加功能，LSP 原则是指导我们如何继承。

在以前写的一篇[里氏替换原则](https://my.oschina.net/liufq/blog/1799960) 的文章里，我提到过：

    每个类都会有public方法，有些类会实现interface，供其他类使用，自身就处在一个服务的位置上。
    每个public方法都是自身所做出的一个承诺，只要你按照要求调用，就会提供正确的服务。
    子类在继承后，固然是获得了超类的带来的‘财富’，更重要的是要遵守超类做出的承诺，
    破坏了这个承诺实际上是没有资格继承超类的。

如果破坏了继承原则，那么开闭原则也就无法使用。子类不按照契约设定编码，那就是在给使用者挖坑。
    
## 实践

需求要求设计一个鸟的继承体系，如下是我们设计的抽象基类：
```java
public abstract class Bird {
    private String name;
    public void setName(String name){
        this.name = name;
    }
    public void fly() {
        System.out.println(name + " fly");
    }
}

```

大部分鸟在这个基类中都工作的很好，但是有一天来了一只企鹅，企鹅是不会飞的，因此我们重写 `fly` 方法
```java
public class Penguin {
    @Override
    public void fly() {
        throw new RuntimeException();
    }
}
```

由于企鹅不会飞，在 `fly` 方法里直接抛出了异常。

注意，这里已经违反了 LSP 原则，在基类中并没有异常抛出，使用方正常使用，而在 `Penguin` 类中 `fly` 方法抛出了异常，违反了基类遵守的契约。

要解决这个问题，我们需要应用单一职责原则来拆分 `Bird` 类，由 `Penguin` 来看， `fly` 功能并不是 `Bird` 承担的职责，应该将其单独放到一个接口中，会飞的鸟自行实现。如果像上面那样，大部分鸟都有一个默认的飞行实现，则我们可以做一个默认的飞行实现类，使用组合的方式放到会飞的鸟中。

```java
public abstract class Bird {
    private String name;
    public void setName(String name){
        this.name = name;
    }
}


public interface Flyable {
    public void fly();
}
```

## 总结

里氏替换原则是继承需要遵循的原则，有时我们可能在无意中就已经违反了原则要求，一是因为我们没有意识到，二是我们设计的接口、抽象基类有问题。遇到违反 LSP 原则的继承，有两招来解决：1. 修改实现，2。 更改设计。
