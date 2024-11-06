---
sort: 1
---

# Interview

1. 设计模式的六大原则

- 开闭原则：对扩展开放，对修改关闭。创建型模式、结构型模式、行为型模式都遵循开闭原则。
- 单一职责原则：一个类只负责一项职责。创建型模式、结构型模式、行为型模式都遵循单一职责原则。
- 依赖倒置原则：高层模块不应该依赖低层模块，二者都应该依赖抽象。创建型模式、结构型模式、行为型模式都遵循依赖倒置原则。
- 接口隔离原则：使用多个专门的接口，而不使用单一的总接口。结构型模式、行为型模式都遵循接口隔离原则。
- 迪米特法则：一个对象应该对其他对象保持最少的了解。结构型模式、行为型模式都遵循迪米特法则。
- 组合/聚合复用原则：尽量使用组合/聚合关系来取代继承关系。结构型模式、行为型模式都遵循组合/聚合复用原则。



2. 设计模式的七大类型

- 创建型模式：适用于对象的创建过程，如工厂模式、抽象工厂模式、单例模式、建造者模式、原型模式。
- 结构型模式：适用于类或对象间的组合关系，如适配器模式、桥接模式、组合模式、装饰器模式、外观模式、享元模式。
- 行为型模式：适用于对象间的交互关系，如策略模式、模板方法模式、观察者模式、迭代器模式、责任链模式、命令模式、状态模式。

3. 设计模式的优缺点

- 优点：
  - 1. 代码可读性高：代码可读性高，容易理解，便于维护。
  - 2. 代码复用性高：代码复用性高，可以提高开发效率。
  - 3. 容易扩展：设计模式易于扩展，可以适应变化。
  - 4. 降低耦合度：降低了代码的耦合度，提高了代码的可维护性。
- 缺点：
  - 1. 设计模式过度使用：过度使用设计模式，会导致代码的复杂性，降低代码的可读性。


## 面试题

### 1. 「观察者模式」与「发布/订阅模式」的区别？

观察者模式是一种软件设计模式，主要针对代码层次，指定类与类之间的组织关系。
发布/订阅模式是一种系统架构模式，主要针对组件/服务层次，指定它们之间的消息传递模式。


以下是 C++ 中五个最常用的设计模式：

**一、单例模式（Singleton Pattern）**

1. **定义和作用**：
   - 单例模式确保一个类只有一个实例，并提供一个全局访问点来访问这个唯一实例。
   - 在一些场景中，比如全局配置管理、日志系统、数据库连接等，只需要一个实例来管理资源或提供全局服务。

2. **实现方式**：
   - 通常通过将构造函数私有化，防止外部直接实例化该类。提供一个静态方法来获取唯一实例，第一次调用该方法时创建实例，后续调用直接返回已创建的实例。
   - 例如：
```cpp
class Singleton {
private:
    static Singleton* instance;
    Singleton() {}
public:
    static Singleton* getInstance() {
        if (instance == nullptr) {
            instance = new Singleton();
        }
        return instance;
    }
};

Singleton* Singleton::instance = nullptr;
```

**二、工厂模式（Factory Pattern）**

1. **定义和作用**：
   - 工厂模式定义了一个用于创建对象的接口，让子类决定实例化哪一个类。它将对象的创建和使用分离，使得代码更加灵活和可维护。
   - 当需要根据不同的条件创建不同类型的对象时，可以使用工厂模式，避免在客户端代码中直接使用具体的构造函数。

2. **实现方式**：
   - 可以创建一个抽象工厂类，定义创建对象的接口。然后，根据具体需求实现不同的具体工厂类，每个具体工厂类负责创建一种特定类型的对象。
   - 例如：
```cpp
class Product {
public:
    virtual void operation() = 0;
};

class ConcreteProductA : public Product {
public:
    void operation() override {
        std::cout << "ConcreteProductA operation" << std::endl;
    }
};

class ConcreteProductB : public Product {
public:
    void operation() override {
        std::cout << "ConcreteProductB operation" << std::endl;
    }
};

class Factory {
public:
    virtual Product* createProduct() = 0;
};

class ConcreteFactoryA : public Factory {
public:
    Product* createProduct() override {
        return new ConcreteProductA();
    }
};

class ConcreteFactoryB : public Factory {
public:
    Product* createProduct() override {
        return new ConcreteProductB();
    }
};
```

**三、观察者模式（Observer Pattern）**

1. **定义和作用**：
   - 观察者模式定义了一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖它的对象都会得到通知并自动更新。
   - 在很多场景中，比如图形用户界面中的事件处理、消息推送系统等，需要实现对象之间的松散耦合，使得一个对象的变化能够自动通知其他相关对象。

2. **实现方式**：
   - 通常包括抽象主题（被观察的对象）和抽象观察者两个接口。具体主题实现抽象主题接口，维护一个观察者列表，并在状态变化时通知所有观察者。具体观察者实现抽象观察者接口，接收主题的通知并进行相应的处理。
   - 例如：
```cpp
class Observer {
public:
    virtual void update() = 0;
};

class Subject {
public:
    virtual void attach(Observer* observer) = 0;
    virtual void detach(Observer* observer) = 0;
    virtual void notify() = 0;
};

class ConcreteSubject : public Subject {
private:
    std::vector<Observer*> observers;
    int state;
public:
    void attach(Observer* observer) override {
        observers.push_back(observer);
    }
    void detach(Observer* observer) override {
        auto it = std::find(observers.begin(), observers.end(), observer);
        if (it!= observers.end()) {
            observers.erase(it);
        }
    }
    void notify() override {
        for (Observer* observer : observers) {
            observer->update();
        }
    }
    void setState(int newState) {
        state = newState;
        notify();
    }
    int getState() const {
        return state;
    }
};

class ConcreteObserver : public Observer {
private:
    int observerState;
    ConcreteSubject* subject;
public:
    ConcreteObserver(ConcreteSubject* subject) : subject(subject) {
        subject->attach(this);
    }
    void update() override {
        observerState = subject->getState();
        std::cout << "Observer state updated to " << observerState << std::endl;
    }
};
```

**四、装饰器模式（Decorator Pattern）**

1. **定义和作用**：
   - 装饰器模式动态地给一个对象添加一些额外的职责，它比继承更加灵活。
   - 当需要在不修改原有对象代码的情况下，为对象添加新的功能时，可以使用装饰器模式。

2. **实现方式**：
   - 包括抽象组件、具体组件、抽象装饰器和具体装饰器几个部分。抽象组件定义了对象的接口，具体组件实现这个接口。抽象装饰器继承自抽象组件，并包含一个指向抽象组件的指针。具体装饰器实现具体的装饰功能，通过调用被装饰对象的方法和添加额外的功能来实现装饰。
   - 例如：
```cpp
class Component {
public:
    virtual void operation() = 0;
};

class ConcreteComponent : public Component {
public:
    void operation() override {
        std::cout << "ConcreteComponent operation" << std::endl;
    }
};

class Decorator : public Component {
protected:
    Component* component;
public:
    Decorator(Component* component) : component(component) {}
    void operation() override {
        if (component) {
            component->operation();
        }
    }
};

class ConcreteDecoratorA : public Decorator {
public:
    ConcreteDecoratorA(Component* component) : Decorator(component) {}
    void operation() override {
        Decorator::operation();
        addedBehavior();
    }
private:
    void addedBehavior() {
        std::cout << "Added behavior in ConcreteDecoratorA" << std::endl;
    }
};
```

**五、策略模式（Strategy Pattern）**

1. **定义和作用**：
   - 策略模式定义了一系列算法，将每个算法封装起来，并且使它们可以相互替换。策略模式让算法的变化独立于使用算法的客户。
   - 在一些场景中，比如排序算法的选择、支付方式的选择等，可以根据不同的情况动态地选择不同的算法。

2. **实现方式**：
   - 包括抽象策略、具体策略和上下文三个部分。抽象策略定义了算法的接口，具体策略实现具体的算法，上下文包含一个指向抽象策略的指针，并根据需要调用具体策略的方法。
   - 例如：
```cpp
class Strategy {
public:
    virtual int execute(int a, int b) = 0;
};

class ConcreteStrategyAdd : public Strategy {
public:
    int execute(int a, int b) override {
        return a + b;
    }
};

class ConcreteStrategySubtract : public Strategy {
public:
    int execute(int a, int b) override {
        return a - b;
    }
};

class Context {
private:
    Strategy* strategy;
public:
    Context(Strategy* strategy) : strategy(strategy) {}
    int executeStrategy(int a, int b) {
        return strategy->execute(a, b);
    }
    void setStrategy(Strategy* newStrategy) {
        if (strategy) {
            delete strategy;
        }
        strategy = newStrategy;
    }
};
```

以下是另外五个在 C++ 中常用的设计模式：

**一、模板方法模式（Template Method Pattern）**

1. **定义和作用**：
   - 模板方法模式定义了一个操作中的算法骨架，将一些步骤延迟到子类中实现。它使得子类可以在不改变算法结构的情况下，重新定义算法中的某些特定步骤。
   - 适用于当多个算法具有相似的结构，但某些具体步骤有所不同的情况。可以将通用的算法框架提取到父类中，而将可变的部分留给子类实现。

2. **实现方式**：
   - 在父类中定义一个模板方法，该方法包含了算法的骨架，调用一系列的抽象方法（这些抽象方法将由子类实现）。子类继承父类，并实现这些抽象方法，从而定制算法的特定步骤。
   - 例如：
```cpp
class AbstractClass {
public:
    void templateMethod() {
        step1();
        step2();
        step3();
    }
    virtual void step1() = 0;
    virtual void step2() = 0;
    virtual void step3() = 0;
};

class ConcreteClassA : public AbstractClass {
public:
    void step1() override {
        std::cout << "ConcreteClassA step1" << std::endl;
    }
    void step2() override {
        std::cout << "ConcreteClassA step2" << std::endl;
    }
    void step3() override {
        std::cout << "ConcreteClassA step3" << std::endl;
    }
};
```

**二、命令模式（Command Pattern）**

1. **定义和作用**：
   - 将请求封装成对象，从而可以将不同的请求进行参数化、排队或者记录请求日志，以及支持可撤销的操作。
   - 在图形用户界面中，命令模式可以用于实现菜单项、按钮等的操作。在需要对操作进行记录、撤销和重做等功能的系统中，命令模式非常有用。

2. **实现方式**：
   - 定义一个抽象命令类，包含一个执行方法。具体命令类实现这个抽象命令类，将一个接收者对象绑定到命令中，并在执行方法中调用接收者的相应方法。调用者持有命令对象，并在需要时调用命令的执行方法。
   - 例如：
```cpp
class Receiver {
public:
    void action() {
        std::cout << "Receiver action" << std::endl;
    }
};

class Command {
public:
    virtual void execute() = 0;
};

class ConcreteCommand : public Command {
private:
    Receiver* receiver;
public:
    ConcreteCommand(Receiver* receiver) : receiver(receiver) {}
    void execute() override {
        receiver->action();
    }
};

class Invoker {
private:
    Command* command;
public:
    void setCommand(Command* command) {
        this->command = command;
    }
    void invoke() {
        command->execute();
    }
};
```

**三、迭代器模式（Iterator Pattern）**

1. **定义和作用**：
   - 提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露该对象的内部表示。
   - 当需要遍历一个复杂的数据结构，如链表、树、图等，而又不想暴露其内部实现细节时，可以使用迭代器模式。

2. **实现方式**：
   - 定义一个抽象迭代器接口，包含一些基本的遍历操作，如获取下一个元素、判断是否还有更多元素等。具体的聚合对象实现一个创建迭代器的方法，返回一个具体的迭代器对象。具体迭代器实现抽象迭代器接口，负责遍历聚合对象中的元素。
   - 例如：
```cpp
class Aggregate {
public:
    virtual int getSize() = 0;
    virtual int getElement(int index) = 0;
    virtual Iterator* createIterator() = 0;
};

class ConcreteAggregate : public Aggregate {
private:
    std::vector<int> data;
public:
    int getSize() override {
        return data.size();
    }
    int getElement(int index) override {
        return data[index];
    }
    Iterator* createIterator() override {
        return new ConcreteIterator(this);
    }
};

class Iterator {
public:
    virtual int next() = 0;
    virtual bool hasNext() = 0;
};

class ConcreteIterator : public Iterator {
private:
    ConcreteAggregate* aggregate;
    int currentIndex;
public:
    ConcreteIterator(ConcreteAggregate* aggregate) : aggregate(aggregate), currentIndex(0) {}
    int next() override {
        return aggregate->getElement(currentIndex++);
    }
    bool hasNext() override {
        return currentIndex < aggregate->getSize();
    }
};
```

**四、代理模式（Proxy Pattern）**

1. **定义和作用**：
   - 为其他对象提供一种代理以控制对这个对象的访问。代理模式可以在不改变目标对象的情况下，为目标对象添加额外的功能，或者控制对目标对象的访问。
   - 例如，可以使用代理模式来实现远程对象的访问、权限控制、缓存等功能。

2. **实现方式**：
   - 定义一个抽象主题接口，目标对象和代理对象都实现这个接口。代理对象持有一个目标对象的引用，并在实现接口方法时，根据需要调用目标对象的方法或者添加额外的功能。
   - 例如：
```cpp
class Subject {
public:
    virtual void request() = 0;
};

class RealSubject : public Subject {
public:
    void request() override {
        std::cout << "RealSubject request" << std::endl;
    }
};

class Proxy : public Subject {
private:
    RealSubject* realSubject;
public:
    Proxy() {
        realSubject = new RealSubject();
    }
    void request() override {
        // 添加额外的功能，如权限检查、日志记录等
        std::cout << "Proxy checking before forwarding request" << std::endl;
        realSubject->request();
    }
};
```

**五、外观模式（Facade Pattern）**

1. **定义和作用**：
   - 为子系统中的一组接口提供一个一致的界面，简化了子系统的使用。外观模式隐藏了子系统的复杂性，提供了一个更简单、更易于使用的接口给客户端。
   - 当一个系统的子系统非常复杂，客户端需要与多个子系统进行交互时，可以使用外观模式来提供一个统一的接口，降低客户端与子系统之间的耦合度。

2. **实现方式**：
   - 创建一个外观类，该类包含了对子系统中各个对象的引用，并提供一些简单的方法来调用子系统中的复杂功能。客户端只需要与外观类进行交互，而不需要了解子系统的内部结构。
   - 例如：
```cpp
class SubsystemA {
public:
    void operationA() {
        std::cout << "SubsystemA operation" << std::endl;
    }
};

class SubsystemB {
public:
    void operationB() {
        std::cout << "SubsystemB operation" << std::endl;
    }
};

class Facade {
private:
    SubsystemA* subsystemA;
    SubsystemB* subsystemB;
public:
    Facade() {
        subsystemA = new SubsystemA();
        subsystemB = new SubsystemB();
    }
    void operation() {
        subsystemA->operationA();
        subsystemB->operationB();
    }
};
```