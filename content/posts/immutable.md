+++
date = '2025-09-04T17:10:09+08:00'
draft = false
title = '并发编程-不变性模式'
tags = ['Java']

+++

# Java不变性模式

不变性模式通过创建不可变对象来避免并发问题。不可变对象一旦创建就不能修改，天然线程安全。

## 不可变对象的特征

- **状态不可变：**对象创建后状态不能改变
- **所有字段都是final：**确保字段不能被重新赋值
- **没有setter方法：**不提供修改状态的方法
- **类是final的：**防止子类破坏不变性

举例说明

```java
public final class ImmutablePerson {
    private final String name;
    private final int age;
    private final List<String> hobbies;
    
    public ImmutablePerson(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        // 防御性复制，避免外部修改
        this.hobbies = Collections.unmodifiableList(new ArrayList<>(hobbies));
    }
    
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    public List<String> getHobbies() {
        // 返回不可修改的视图
        return hobbies;
    }
    
    // 修改操作返回新对象
    public ImmutablePerson withAge(int newAge) {
        return new ImmutablePerson(this.name, newAge, 
            new ArrayList<>(this.hobbies));
    }
}
```



## 使用Builder模式创建不可变对象

```java
public final class ImmutablePersonBuilder {
    private final String name;
    private final int age;
    private final List<String> hobbies;
    
    private ImmutablePersonBuilder(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.hobbies = Collections.unmodifiableList(builder.hobbies);
    }
    
    public static class Builder {
        private String name;
        private int age;
        private List<String> hobbies = new ArrayList<>();
        
        public Builder setName(String name) {
            this.name = name;
            return this;
        }
        
        public Builder setAge(int age) {
            this.age = age;
            return this;
        }
        
        public Builder addHobby(String hobby) {
            this.hobbies.add(hobby);
            return this;
        }
        
        public ImmutablePersonBuilder build() {
            return new ImmutablePersonBuilder(this);
        }
    }
    
    // getter方法...
}
```

