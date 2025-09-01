+++
date = '2025-09-01T17:11:44+08:00'
draft = false
title = 'Java_generics'
featuredImagePreview = "generics.webp"
+++

# 什么是泛型
先看Java标准库中的`ArrayList` ，这是一个“可变长”数组。内部通过`Object[]`数组实现
```java
public class ArrayList {
    private Object[] array;
    private int size;
    public void add(Object e) {...}
    public void remove(int index) {...}
    public Object get(int index) {...}
}
```

在很久很久以前（泛型之前）如果用ArrayList存储String类型，需要强制转型
```java
ArrayList list = new ArrayList();
list.add("Hello");
// 获取到Object，必须强制转型为String:
String first = (String) list.get(0);
```
后来出现了泛型，可以把`ArrayList`变成一种模板`ArrayList<T>` 编写一次模版，可以创建任意类型的`ArrayList`：
```java
public class ArrayList<T> {
    private T[] array;
    private int size;
    public void add(T e) {...}
    public void remove(int index) {...}
    public T get(int index) {...}
}
// 创建可以存储String的ArrayList:
ArrayList<String> strList = new ArrayList<String>();
// 创建可以存储Float的ArrayList:
ArrayList<Float> floatList = new ArrayList<Float>();
// 创建可以存储Person的ArrayList:
ArrayList<Person> personList = new ArrayList<Person>();
```

## 向上转型
在Java标准库中的`ArrayList<T>`实现了`List<T>`接口，它可以向上转型为`List<T>` 。同时通过`List<String>`定义后 编译器可以自动推断出是`String`类型。因此可以简写
```java
List<String> list = new ArrayList<String>();
// 简写
List<String> list = new ArrayList<>();
```

# 编写泛型
通常来说，泛型类一般用在集合类中。示例如下：可以看到和普通类的区别在于，定义是多了一个`<T>`，变量类型也都是`<T>`
```java
public class Pair<T> {
    private T first;
    private T last;
    public Pair(T first, T last) {
        this.first = first;
        this.last = last;
    }
    public T getFirst() {
        return first;
    }
    public T getLast() {
        return last;
    }
}

```

## 静态方法
编写泛型类时，要特别注意，泛型类型`<T>`不能用于静态方法。原因是这个参数对于类的所有实例方法和成员变量可见，但是静态方法属于类本身，而不属于类的任何实例。静态方法在类加载时就已经存在，早于任何类的实例的创建。所以静态方法不能直接访问类级别的泛型类型参数 `<T>`  正确做法是——在静态方法上单独声明泛型类型参数`<K>`。
```java
public class Pair<T> {
    private T first;
    private T last;
    public Pair(T first, T last) {
        this.first = first;
        this.last = last;
    }
    public T getFirst() { ... }
    public T getLast() { ... }

    // 静态泛型方法应该使用其他类型区分:
    public static <K> Pair<K> create(K first, K last) {
        return new Pair<K>(first, last);
    }
}

```
