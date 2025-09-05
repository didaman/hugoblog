+++
date = '2025-09-04T16:43:44+08:00'
draft = false
title = '并发编程-单例模式'
tags = ['Java']
+++

# Java单例模式

单例模式是最常用的设计模式之一，但在多线程环境下，传统的单例实现可能存在线程安全问题。我们需要掌握各种线程安全的单例实现方式。

## 懒汉式单例

**线程不安全的懒汉式**

```java
public class UnsafeLazySingleton {
    private static UnsafeLazySingleton instance;
    
    private UnsafeLazySingleton() {}
    
    // 线程不安全：多个线程可能同时创建实例
    public static UnsafeLazySingleton getInstance() {
        if (instance == null) {
            instance = new UnsafeLazySingleton();
        }
        return instance;
    }
}
```

**线程安全的懒汉式（同步方法）**

```java
public class SafeLazySingleton {
    private static SafeLazySingleton instance;
    
    private SafeLazySingleton() {}
    
    // 线程安全但性能较差：每次调用都需要同步
    public static synchronized SafeLazySingleton getInstance() {
        if (instance == null) {
            instance = new SafeLazySingleton();
        }
        return instance;
    }
}
```



## 饿汉式单例

饿汉式单例在类加载时就创建实例，天然线程安全，但可能造成资源浪费。

```java
public class EagerSingleton {
    // 类加载时就创建实例，线程安全
    private static final EagerSingleton INSTANCE = new EagerSingleton();
    
    private EagerSingleton() {}
    
    public static EagerSingleton getInstance() {
        return INSTANCE;
    }
}
```

## 双重检查锁定

双重检查锁定（Double-Checked Locking）是一种优化的懒汉式实现，既保证线程安全又提高性能。

```java
public class DoubleCheckedSingleton {
    // 使用volatile确保可见性和禁止指令重排序
    private static volatile DoubleCheckedSingleton instance;
    
    private DoubleCheckedSingleton() {}
    
    public static DoubleCheckedSingleton getInstance() {
        // 第一次检查：避免不必要的同步
        if (instance == null) {
            synchronized (DoubleCheckedSingleton.class) {
                // 第二次检查：确保只创建一个实例
                if (instance == null) {
                    instance = new DoubleCheckedSingleton();
                }
            }
        }
        return instance;
    }
}
```

- `volatile` 关键字：保证多线程环境下变量可见性。同时禁止指令重排
- 构造函数：声明为`private` 让外部类无法通过 `new` 关键字来创建实例。
- `getInstance` 方法：
  - 第一次检查，避免不必要的同步操作
  - `synchronized (DoubleCheckedSingleton.class)` 同步块
  - 第二次检查：因为在同步块外可能有多个线程同时通过了第一次检查，所以进入同步块后需要再次检查 `instance` 是否为 `null`。如果仍然为 `null`，则创建 `DoubleCheckedSingleton` 的实例。

## 静态内部类单例（推荐）

利用类加载机制保证线程安全，同时实现懒加载。

```java
public class StaticInnerClassSingleton {
    private StaticInnerClassSingleton() {}
    
    // 静态内部类，只有在被引用时才会加载
    private static class SingletonHolder {
        private static final StaticInnerClassSingleton INSTANCE = 
            new StaticInnerClassSingleton();
    }
    
    public static StaticInnerClassSingleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

- 构造函数：声明为`private` 让外部类无法通过 `new` 关键字来创建实例。从而保证单例唯一性
- 静态内部类：
  - 定义了一个静态常量 `INSTANCE`，它是 `StaticInnerClassSingleton` 类的实例。同时`final` 关键字确保这个实例一旦被初始化就不能再被重新赋值。
  - 利用静态内部类特点——只有在被引用时才加载。（实现懒加载）
- 获取实例方法`getInstance`：提供给外部的获取类实例的访问点。当外部类调用 `getInstance` 方法时，实际上是获取 `SingletonHolder` 类中的 `INSTANCE` 实例。多线程环境下只会被初始化一次。



# 总结

推荐使用静态内部类单例的方式。简单可靠，不依赖Java版本。



