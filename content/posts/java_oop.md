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

## 构造函数

构造函数是Java中用于初始化对象的特殊方法。在创建类的实例时，会被调用，通常用于初始化对象的状态。构造函数具有以下特点：

- 构造函数名和类名相同
- 构造函数没有返回值
- 如果没有显式指定构造函数，编译器会自动提供一个无参构造函数
- 可以有多个构造函数（构造函数重载）
  - 参数列表必须不同
  - 提供多种对象创造方式
  - 可以用`this()`调用其他构造函数

### 构造函数链式调用

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

### 构造函数参数验证

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

### 构造函数防御性复制

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

### 构造函数设计原则

- 进行参数验证
- 使用构造函数链式调用
- 防御性复制可变对象
- 提供清晰的错误信息
- 保持构造函数简洁





