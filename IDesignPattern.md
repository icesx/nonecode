Design Pattern
=====

### 1工厂模式

直接创建对象

### 2抽象工厂

返回抽象的工厂，由工厂再创建对象

### 3工厂方法

不同的工厂，相同的工厂方法名，生产不同的产品。

### 4构建器模式

隐藏对象构建过程到builder中。

### 5单例模式

一个loader范围的唯一，如果是多个loader加载class的时候，不能保证全局唯一性。

### 6访问者模式

通过visit(this)实现外部对象对this的操作。

### 7观察者模式

Publish/Subscribe

### 8代理模式

代理角色和真实角色是同一类对象



### 9命令模式

command invoker receiver

### 10装饰器模式

使用代理模式，代理和真实对象之间的的关系通常在编译时就已经确定了(代理类中包含一个被代理类的实现)，而装饰者能够在运行时递归地被构造。

装饰器也要实现被装饰对象的接口。

### 11适配器模式

将一个类的接口转换成客户希望的另外一个接口。

对被适配的对象增加功能。

### 12享元模式

享元模式以共享的方式高效地支持大量的细粒度对象。


### 13 不变模式



### 14状态模式



### 15职责连模式



### 16中介者模式

降低对象通讯复杂度，网状通讯转换为树状，中介者负责沟通

### 17门面模式



### 19 迭代器模式



### 20策略模式

封装策略到一个个独立的类中，通过context进行组装调用。

与状态模式和桥接模式类似：



### 21组合模式

jdom

### 22桥接模式

抽象和行为独自变化。
提前设计好规则，将不同的对象传递进去，完成不同的逻辑。
抽象和实现没有固定的绑定，以传参的方式关联。

**将对接和对象行为解耦通过桥接器桥接起来**


### 23模板模式

servlet

### 24 解释器模式

