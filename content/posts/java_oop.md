+++
date = '2025-08-28T14:56:00+08:00'
draft = false
title = 'Java面向对象基础'
tags = ['Java']
featuredImagePreview = "javaoop.png"
+++

# Java面向对象基础

面向对象编程（Object-Oriented Programming，OOP）是一种编程范式。通过把现实中的事物抽象成程序中的对象（Object)来处理逻辑。Java是一门纯面向对象的编程语言。

![Java面向对象](javaoop.png)

# 构造函数

构造函数是Java中用于初始化对象的特殊方法。在创建类的实例时，会被调用，通常用于初始化对象的状态。构造函数具有以下特点：

- 构造函数名和类名相同
- 构造函数没有返回值
- 如果没有显式指定构造函数，编译器会自动提供一个无参构造函数
- 可以有多个构造函数（构造函数重载）
  - 参数列表必须不同
  - 提供多种对象创造方式
  - 可以用`this()`调用其他构造函数

## 构造函数链式调用

如果需要多个构造函数，不同构造函数又很相似，比如设定一个长方形的长和宽，颜色是一个构造函数。还需要一个创造正方形的构造函数，就可以用链式调用。

使用`this()`链式调用构造函数，注意以下规则：

- 位置：必须是新构造函数的第一条
- 参数：使用正确的参数列表
- 复用：把最通用的构造函数集中在一个基本的构造函数



```java
public class Rectangle {
    private double width;
    private double height;
    private String color;
    
    // 最完整的构造函数
    public Rectangle(double width, double height, String color) {
        if (width <= 0 || height <= 0) {
            throw new IllegalArgumentException("Width and height must be positive");
        }
        this.width = width;
        this.height = height;
        this.color = color != null ? color : "white";
    }
    
    // 调用完整构造函数，使用默认颜色
    public Rectangle(double width, double height) {
        this(width, height, "white");
    }
    
    // 创建正方形，调用两参数构造函数
    public Rectangle(double side) {
        this(side, side);
    }
    
    // 默认构造函数，创建单位正方形
    public Rectangle() {
        this(1.0);
    }
}
```

## 构造函数参数验证

在构造函数中进行参数验证是确保对象状态有效性的重要手段。作用如下

- 防止创建无效状态的对象
- 提供清晰的错误信息
- 遵循"快速失败"原则
- 提高代码的健壮性

```java
public class BankAccount {
    private String accountNumber;
    private String ownerName;
    private double balance;
    
    public BankAccount(String accountNumber, String ownerName, double initialBalance) {
        // 验证账号
        if (accountNumber == null || accountNumber.trim().isEmpty()) {
            throw new IllegalArgumentException("Account number cannot be null or empty");
        }
        if (!accountNumber.matches("\\d{10}")) {
            throw new IllegalArgumentException("Account number must be 10 digits");
        }
        
        // 验证姓名
        if (ownerName == null || ownerName.trim().isEmpty()) {
            throw new IllegalArgumentException("Owner name cannot be null or empty");
        }
        if (ownerName.length() > 50) {
            throw new IllegalArgumentException("Owner name cannot exceed 50 characters");
        }
        
        // 验证余额
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Initial balance cannot be negative");
        }
        
        // 设置属性
        this.accountNumber = accountNumber;
        this.ownerName = ownerName.trim();
        this.balance = initialBalance;
    }
}
```

## 构造函数防御性复制

在 Java 编程中，当一个类的构造函数接收可能被外部修改的对象（如 `ArrayList`）作为参数时，为了防止该对象在类外部被修改后影响到类内部的状态，会在构造函数中对该参数对象进行复制，这就是构造函数的防御性复制。保证类的安全性和一致性。

例如，一个表示学生成绩列表的类 `StudentGrades`，构造函数接收一个 `List<Integer>` 作为参数，如果不进行防御性复制，外部代码修改这个 `List` 后，`StudentGrades` 类中的成绩列表也会改变，这显然不符合预期。

对于集合类型（如 `List`、`Set`、`Map`）或可变对象如 `Date` 类（`java.util.Date` 是可变的）需要防御性复制。

```java
import java.util.ArrayList;
import java.util.List;

public class StudentGrades {
    private List<Integer> grades;

    public StudentGrades(List<Integer> grades) {
        this.grades = new ArrayList<>(grades); // 防御性复制
    }

    public List<Integer> getGrades() {
        return new ArrayList<>(grades); // 防御性复制
    }
}
```

- **性能影响**：防御性复制会增加额外的开销，特别是对于大型对象或集合。在性能敏感的场景中，需要权衡使用防御性复制带来的安全性和性能损失。
- **正确选择复制方式**：不同的对象类型可能有不同的复制方式，如 `clone` 方法、构造函数复制等。需要确保选择的复制方式能够真正创建一个独立的副本，避免出现浅拷贝的问题，导致数据仍然存在被外部修改的风险。

## 构造函数设计原则

- 进行参数验证
- 使用构造函数链式调用
- 防御性复制可变对象
- 提供清晰的错误信息
- 保持构造函数简洁

# 继承

继承是面向对象编程的核心特性之一，它允许一个类（子类）获得另一个类（父类）的属性和方法。通过继承，我们可以实现代码复用、扩展功能和建立类之间的层次关系。

`protected`修饰符是专门为继承设计的，它允许子类访问父类的成员，但不允许其他包中的非子类访问。

## 方法重写（Override）

在子类中重新定义父类的方法，叫做方法重写。需要使用`@Override`注解。注意子类重写的方法签名必须和父类的方法完全相同。

```java
@Override
public void makeSound() {
    System.out.println("汪汪汪！");
}
```

常见问题：如何区分**重写（Override）**和重载**重载（Overload）**？

- 重写：子类重新定义父类中的方法，方法名和参数都相同。
- 重载：同一个类中定义多个同名但是参数不同的方法。

### 协变返回类型

**重写后返回类型可以是父类方法返回类型的子类** 这种情况被称为协变返回类型（Covariant Return Types）

```java
class Animal {
    public Animal getInstance() {
        return new Animal();
    }
}

class Dog extends Animal {
    @Override
    public Dog getInstance() {
        return new Dog();
    }
}

public class Main {
    public static void main(String[] args) {
        Animal animal = new Dog();
        Dog dog = animal.getInstance(); // 这里不需要进行类型转换，因为Dog类重写的getInstance方法返回Dog类型
    }
}
```

在上述代码中，`Animal`类有一个`getInstance`方法，返回类型是`Animal`。`Dog`类继承自`Animal`并重写了`getInstance`方法，其返回类型是`Dog`，`Dog`是`Animal`的子类，这是合法的重写。

**使用协变返回类型的好处**：

- **提高代码的灵活性**：允许子类在重写方法时返回更具体的类型，使得调用者可以获得更明确的对象类型，而无需额外的类型转换。
- **符合面向对象的多态特性**：子类在重写方法时能够以更具体的方式来实现行为，包括返回更具体的类型，这与多态的概念相契合



不过需要注意，协变返回类型仅限于**引用类型**：对于基本数据类型，重写方法的返回类型必须与父类方法的返回类型完全相同。例如，如果父类方法返回`int`，子类重写方法不能返回`long`，即使`long`可以容纳`int`的值。同时注意基本类型的包装类如`Long`和`Integer`不是父类和子类关系，不存在继承关系。所以也不能用协变返回类型。



### 重写方法访问修饰符限制

子类重写父类方法时，访问修饰符可以更宽松，但不能更严格，这遵循里氏替换原则。父类可以设置严格的修饰符，防止被随意继承。子类则通过更宽松的访问修饰符确保子类对象能更容易调用方法。符合面向对象编程中关于扩展性的要求。

正确作法：如果父类方法是 `protected`，子类重写该方法时可以使用 `public`

```java
class Parent {
    protected void parentMethod() {
        System.out.println("This is parent method.");
    }
}

class Child extends Parent {
    @Override
    public void parentMethod() {
        System.out.println("This is overridden method in child.");
    }
}
```

编译错误情况：子类更严格的访问限制，破坏类继承体系中方法可访问一致性。

```java
class Parent {
    public void parentMethod() {
        System.out.println("This is parent method.");
    }
}

class Child extends Parent {
    // 以下代码会报错，访问修饰符不能更严格
    // @Override
    // protected void parentMethod() {
    //     System.out.println("This is overridden method in child.");
    // }
}
```



**里氏替换原则简述**：里氏替换原则（Liskov Substitution Principle, LSP）是面向对象设计的基本原则之一，它指出：所有引用基类（父类）的地方必须能透明地使用其子类的对象。也就是说，在程序中如果使用父类对象的地方，替换成子类对象后，程序的行为不会发生改变。在继承中，子类用更宽松的访问限制保证在使用父类对象的地方，替换为子类对象后，子类对象的重写方法依然能被调用。



### 不能被重写的方法

父类中有些方法不能被子类重写。包括

- `private`方法：私有的，对子类不可见
- `static`方法：属于类本身，不是类的实例。重写是基于对象实例的**多态行为**，`static` 方法在编译时就确定了调用版本，不依赖于对象的实际类型，所以不能被重写。
- `final`方法：规定final不能被重写。为了确保某些方法的行为在继承体系中保持不变，防止子类对其进行修改。
- 构造方法：构造方法用于创建对象并初始化对象的状态，它的名称与类名相同，且没有返回类型。每个类都有自己独立的构造方法，子类通过 `super` 关键字调用父类的构造方法来完成父类部分的初始化，但这不是重写的概念。

其中static比较特殊，`static` 方法属于类本身，而不属于类的实例。调用 `static` 方法是通过类名直接调用，而不是通过对象实例调用。虽然不能重写父类static方法但是可以通过定义与父类相同的static方法，实现方法隐藏（method hiding）但这与重写有着本质的区别。

```java
class Parent {
    public static void staticMethod() {
        System.out.println("This is a static method in Parent class.");
    }
}

class Child extends Parent {
    // 这不是重写，而是隐藏
    public static void staticMethod() {
        System.out.println("This is a static method in Child class.");
    }
}
```

- 通过 `Parent.staticMethod()` 调用，执行的是 `Parent` 类中的 `static` 方法；
- 通过 `Child.staticMethod()` 调用，执行的是 `Child` 类中的 `static` 方法。



## 抽象类

抽象类通常用于定义一组**相关类的通用特征和行为**，同时强制子类实现特定的方法。用关键词`abstract`修饰，它不能被直接实例化。特点是：

- **不能实例化：**不能使用new关键字创建抽象类的对象
- **可以有构造方法：**用于初始化抽象类中的成员变量
- **可以包含具体方法：**提供通用的实现逻辑
- **可以包含抽象方法：**强制子类提供具体实现
- **可以有成员变量：**存储对象的状态信息

### 抽象类 vs 接口

| 特性       | 抽象类         | 接口                                    |
| :--------- | :------------- | :-------------------------------------- |
| 关键字     | abstract       | interface                               |
| 继承       | 单继承         | 多实现                                  |
| 方法       | 可以有具体方法 | 默认抽象方法（Java 8+可有**默认方法**） |
| 变量       | 可以有实例变量 | 只能有常量                              |
| 构造方法   | 可以有         | 没有                                    |
| 访问修饰符 | 任意           | public                                  |



# 多态

多态（Polymorphism）是面向对象编程的三大特性之一。多态是指对于同一个操作作用于不同的对象时，可以有不同的解释，产生不同的执行结果。多态主要通过方法重写和动态绑定来实现。

核心思想是："同一个接口，不同的实现"。通过父类引用指向子类对象，在运行时根据对象的实际类型来决定调用哪个方法。

动态绑定是指在运行时根据对象的实际类型来决定调用哪个方法，这是多态的核心机制。

```java
// 动态绑定演示
Animal[] animals = {
    new Dog("大黄", 4, "土狗"),
    new Cat("小白", 3, "白色"),
    new Bird("小红", 2, true),
    new Dog("小黑", 2, "拉布拉多"),
    new Cat("花花", 1, "花色")
};

// 同一个循环，不同的行为
for (Animal animal : animals) {
    animal.makeSound();  // 运行时决定调用哪个类的方法
    animal.move();       // 每种动物都有不同的移动方式
}
```



# 封装

封装是面向对象编程的三大特性之一（封装、继承、多态）。封装是指将对象的状态信息隐藏在对象内部，不允许外部程序直接访问对象内部信息，而是通过该类所提供的方法来实现对内部信息的操作和访问。

核心思想是**"隐藏实现细节，只暴露必要的接口"**。这样可以保护对象的内部状态，防止外部代码的不当访问和修改，同时提高代码的安全性和可维护性。



