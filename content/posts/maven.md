+++
date = '2025-09-10T17:04:42+08:00'
draft = false
title = 'Maven'
tags = ['Java']
+++

# Maven
Maven安装直接参考 https://developer.aliyun.com/article/1078699

# Maven介绍
用命令创建一个新Maven项目
```
mvn archetype:generate -DgroupId=com.example -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false 
```

- `DgroupId=com.example`：项目的组织标识符（通常是公司或组织的域名倒序）
- `DartifactId=my-app`：项目名称（即生成的项目文件夹名称）
- `DarchetypeArtifactId=maven-archetype-quickstart`：使用 Maven 提供的基础项目模板
- `DinteractiveMode=false`：非交互模式，自动创建项目。

生成目录结构：
```
my-app
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── com
    │           └── example
    │               └── App.java
    │   └── resources
    └── test
        └── java
        │   └── com
        │       └── example
        │           └── AppTest.java
        └── resources
```

- `pom.xml`是项目描述文件
- `src/main/java` 存放Java源码
- `src/main/resources` 存放资源文件
- `src/test/resources` 存放测试资源

pom.xml内容如下
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>my-app</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>my-app</name>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
    <java.version>17</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>2.0.17</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.5.17</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>3.0.0</version>
        <configuration>
          <executable>java</executable>
          <mainClass>com.example.App</mainClass>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>

```

# 构建流程
Maven有一套标准化的构建流程。可以实现自动化编译、打包、发布等功能。

## Lifecycle和Phase
完整一整套构建流程就是声明周期Lifecycle，Maven的生命周期由一些阶段（phase）构成。`phase`（阶段）是这个流程中按顺序执行的 “具体步骤”。

Maven 有 3 个最核心的生命周期，就像 “做饭”“洗衣服”“打扫卫生” 是不同的生活流程：
- **clean**：负责 “清理项目”（比如删除编译产生的临时文件）
- **default**：负责 “构建项目”（最常用，包含编译、测试、打包等核心步骤）
- **site**：负责 “生成项目文档”（比如生成 HTML 格式的说明文档）

每个生命周期都包含一些固定顺序的步骤，称为phase。比如default生命周期，包含的关键phase顺序：

- `validate`：验证项目是否完整（比如检查配置文件是否存在）
- `compile`：编译源代码（把`.java`变成`.class`）
- `test`：运行测试代码（比如执行 JUnit 测试）
- `package`：把代码打包（比如变成`.jar`或`.war`文件）
- `install`：把打好的包放到本地仓库（供自己电脑上的其他项目使用）
- `deploy`：把包上传到远程仓库（供团队其他人使用）

运行规则——默认执行`default`生命周期，从开始执行到`package`为止。

```cmd
mvn package
```

实际开发常用命令

```cmd
mvn clean // 清理所有生成的class和jar
mvn clean // compile：先清理，再执行到compile
mvn clean // test：先清理，再执行到test，因为执行test前必须执行compile，所以这里不必指定compile；
mvn clean // package：先清理，再执行到package。
```

经常用到的phase其实只有几个：

- clean：清理
- compile：编译
- test：运行测试
- package：打包

## Goal

执行一个phase又会出发一个或多个goal（就像一个class下包含多个method）

| 执行的Phase | 对应执行的Goal                     |
| ----------- | ---------------------------------- |
| compile     | compiler:compile                   |
| test        | compiler:testCompile surefire:test |

# Maven插件

Maven执行每个phase都是通过某个插件（plugin）执行的。比如执行`compile`时，会找到`compiler`插件，执行`compiler:compile`这个goal完成编译。

Maven内置了常用的标准插件：

| 插件名称 | 对应执行的phase |
| -------- | --------------- |
| clean    | clean           |
| compiler | compile         |
| surefire | test            |
| jar      | package         |



## 其他插件

内置插件不能满足要求时，还可用一些自定义插件。

### exec-maven-plugin

`exec-maven-plugin` 主要作用是**在 Maven 构建过程中执行外部程序或 Java 代码**。它能帮你在 Maven 命令中直接运行 Java 主类、脚本（如 Shell、Python）或系统命令，无需手动敲 `java` 命令或脚本命令。

它最常用的场景是：

- 直接运行 Java 项目的主类（开发阶段快速测试）
- 执行系统命令（如删除文件、调用脚本）
- 在 Maven 生命周期的某个阶段（如 `compile` 之后）自动执行指定操作

插件包含两个goal：`java`和`exec`

1. **`exec:java`**
   直接运行 Java 类（需指定主类全路径），无需先打包成 JAR。
   特点：会自动依赖项目的 classpath（即项目的依赖包和编译后的类），适合运行项目内的 Java 程序。
2. **`exec:exec`**
   执行外部程序或系统命令（如 `ls`、`python script.py`、`echo "hello"` 等）。
   特点：更灵活，可调用任何系统可执行的命令，但需要手动处理类路径（如果运行 Java 程序）。

**使用方法**

例1 运行Java主类

在pom.xml中配置插件如下

```xml
<build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>3.0.0</version>
        <configuration>
          <executable>java</executable>
          <mainClass>com.example.App</mainClass>
          <!-- 可选：传递给主类的参数 -->
          <arguments>
            <argument>arg1</argument>
            <argument>arg2</argument>
          </arguments>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

配置后，在命令行执行：

```cmd
mvn exec:java
```

例2 用 `exec:exec` 执行系统命令

比如在构建时执行 `echo "构建完成"` 命令

```xml
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>3.1.0</version>
  <configuration>
    <!-- 命令：Windows 用 cmd /c echo，Linux/macOS 用 /bin/sh -c echo -->
    <executable>cmd</executable> <!-- Windows 系统 -->
    <!-- <executable>/bin/sh</executable> --> <!-- Linux/macOS 系统 -->
    <arguments>
      <argument>/c</argument> <!-- 配合 cmd 使用 -->
      <!-- <argument>-c</argument> --> <!-- 配合 sh 使用 -->
      <argument>echo "构建完成！"</argument>
    </arguments>
  </configuration>
</plugin>
```

执行命令：

```cmd
mvn exec:exec
```

会在控制台输出 `构建完成！`。

# Maven常用命令

## 指定setting.xml
```
mvn install --settings c:\user\settings.xml 
```

运行时指定setting.xml
```
mvn spring-boot:run -Dspring-boot.run.profiles=dev --settings settings.xml
```