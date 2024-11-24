---
title: "Web组 第五次授课"
description: "异常处理、IO、多线程、Maven及常用的库"
date: "Nov 17 2024"
---

## 反射及其应用、SpringBoot 基础

## 反射

在 Java 中，反射是一种强大的机制，**它允许程序在运行时检查或修改其自身行为**。使用反射，程序能够访问类的属性和方法，即使在编译时这些类的名称并未明确给出。这使得 Java 程序可以在运行时动态地创建对象、调用方法、改变字段等，即便这些类、方法或字段在编写原始代码时不可知。

```java
// 举例：输出一个类所有的字段和方法
public class Main {

    int field1;

    void method1() {
    }

    void method2() {
    }

    public static void main(String[] args) throws InterruptedException, Exception {
        Class<?> clazz = Main.class;
        for (Field field : clazz.getDeclaredFields()) {
            System.out.println("Field: " + field.getName());
        }
        for (Method method : clazz.getDeclaredMethods()) {
            System.out.println("Method: " + method.getName());
        }
    }
}
```

```java
// 举例：调用一个方法
Class<?> clazz = Main.class;
Constructor<?> constructor = clazz.getConstructor();
Object instance = constructor.newInstance();
clazz.getDeclaredMethod("method1").invoke(instance);
```

Class 类中用于获取构造方法的方法

| 方法名                                                         | 描述                           |
| -------------------------------------------------------------- | ------------------------------ |
| Constructor<?>[] getConstructors()                             | 返回所有公共构造方法对象的数组 |
| Constructor<?>[] getDeclaredConstructors()                     | 返回所有构造方法对象的数组     |
| Constructor getConstructor(Class<?>... parameterTypes)         | 返回单个公共构造方法对象       |
| Constructor getDeclaredConstructor(Class<?>... parameterTypes) | 返回单个构造方法对象           |

Constructor 类中用于创建对象的方法

| 方法名                            | 描述                         |
| --------------------------------- | ---------------------------- |
| T newInstance(Object... initargs) | 根据指定的构造方法创建对象   |
| void setAccessible(boolean flag)  | 设置为 true,表示取消访问检查 |

**反射获取成员变量**

| 方法名                                  | 描述                           |
| --------------------------------------- | ------------------------------ |
| Field[] getFields()                     | 返回所有公共成员变量对象的数组 |
| Field[] getDeclaredFields()             | 返回所有成员变量对象的数组     |
| Field getField(String name)             | 返回单个公共成员变量对象       |
| Field getDeclaredField(String name)     | 返回单个成员变量对象           |
| void set(Object instance, Object value) | 给指定对象的成员变量赋值       |
| Object get(Object instance)             | 返回成员变量的值               |

### 注解

Java 注解（Annotations）是从 Java 5 开始引入的一种元数据形式，它们提供了一种为代码添加信息的方法，但不直接影响代码的执行。注解可以被用于类、方法、变量、参数和 Java 包等。使用注解的主要目的是提供信息给编译器，进行代码分析，或者通过运行时反射机制实现特定功能。

如何定义一个注解？

```java
public @interface MyAnnotation {
    String value() default "默认值";
}
```

如何使用注解？

```java
@MyAnnotation(value = "example")
public class MyClass {
}
```

### 作用？

Spring 框架的实现

- Spring 框架的核心功能之一是依赖注入，它允许组件（beans）之间的依赖关系通过配置方式来管理，而不是通过组件内部硬编码。Spring 通过反射实现了这一机制。
- Spring AOP（面向切面编程）允许开发者定义方法拦截器和切点，来统一处理诸如事务管理、安全检查、日志记录等横切关注点。在运行时，使用反射调用方法前后，可以插入额外的操作。

### 缺点

- 性能开销：由于反射涉及动态解析的类型，因此无法执行某些 Java 虚拟机优化。 因此，反射操作的性能要比非反射操作的性能要差，应该在性能敏感的应用程序中频繁调用的代码段中避免。
- 破坏封装性：反射调用方法时可以忽略权限检查，因此可能会破坏封装性而导致安全问题。
- 内部曝光：由于反射允许代码执行在非反射代码中非法的操作，例如访问私有字段和方法，所以反射的使用可能会导致意想不到的副作用，这可能会导致代码功能失常并可能破坏可移植性。

## SQL

SQL（Structured Query Language）是一种用于访问和操作关系数据库的标准编程语言。它允许用户定义数据的结构，操纵数据，以及对数据进行查询。

### 功能

1. **数据查询**：使用`SELECT`语句从一个或多个表中查询数据。
2. **数据操作**：包括插入（`INSERT`）、更新（`UPDATE`）、删除（`DELETE`）数据。
3. **数据定义**：使用`CREATE`, `ALTER`, `DROP`等语句创建、修改、删除表和数据库架构。
4. **数据控制**：通过`GRANT`和`REVOKE`语句控制不同用户和角色对数据的访问权限。
5. **事务管理**：使用`BEGIN`, `COMMIT`, `ROLLBACK`等语句管理事务，保证数据的一致性和完整性。

SQL 有标准化的语法和操作，但不同的数据库系统往往会有自己的扩展（称为方言），这可能导致在不同系统间迁移或者执行相同任务时需要调整 SQL 代码。

### 数据库的分类

- 关系型数据库（Relational Database Management System, RDBMS）：关系型数据库指的是使用关系模型（二维表格模型）来组织数据的数据库。
- 非关系型数据库（Not Only SQL, NoSQL）：意为不仅仅是 SQL。通常指数据以对象的形式存储在数据库中，而对象之间的关系通过每个对象自身的属性来决定，常用于存储非结构化的数据。

## MySQL

### 安装

打开https://www.mysql.com
![1](/mysql_1.png)
![2](/mysql_2.png)
![3](/mysql_3.png)
![4](/mysql_4.png)
![5](/mysql_5.png)
![6](/mysql_6.png)
![7](/mysql_7.png)
![8](/mysql_8.png)
![9](/mysql_9.png)
![10](/mysql_10.png)
![11](/mysql_11.png)
![12](/mysql_12.png)
![13](/mysql_13.png)
![14](/mysql_14.png)
![15](/mysql_15.png)
![16](/mysql_16.png)
![17](/mysql_17.png)
![18](/mysql_18.png)
![19](/mysql_19.png)

### 基本命令、语法

连接到数据库

```bash
mysql -u root -p 密码
```

查看所有数据库

```mysql
show databases;
```

创建一个数据库

```mysql
create database 名字;
```

选择一个数据库

```mysql
use 库名;
```

显示库中的所有表

```mysql
show tables;

show tables from 库名;
```

创建一个表

```mysql
create table 表名
(
  字段名 类型 参数,
  ...,
  其它条件
);

create table students
(
  id int auto_increment,
  student_id varchar(255),
  name varchar(255),
  primary key (id)
);
```

向表中添加一行数据

```mysql
insert into 表名 (字段名1, 字段名2, ...) values (字段1内容, 字段2内容, ...)

insert into students (student_id, name) values ('B24xxxxxx', '南邮校科协')
```

查询一行数据

```mysql
select * from 表名 where 条件

select * from students where student_id = 'B24xxxxxx'

select name from students where student_id = 'B24xxxxxx'
```

更新一行数据

```mysql
update 表名 set 字段名1 = 字段内容1, 字段名2 = 字段内容2, ... where 条件

update students set name = '张三' where student_id = 'B24xxxxxx'
```

删除一行数据

```mysql
delete from 表名 where 条件

delete from students where student_id = 'B24xxxxxx'
```

### 如何在 Java 中使用 MySQL？

首先，在 Maven 中导入 mysql-connector-j 依赖

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.1.0</version>
</dependency>
```

加载驱动

```java
try {
    Class.forName("com.mysql.cj.jdbc.Driver");
} catch (ClassNotFoundException e) {
    System.out.println("MySQL JDBC driver not found.");
    e.printStackTrace();
}
```

获取数据库连接

```java
String url = "jdbc:mysql://localhost:3306/mydatabase?useSSL=false";
String user = "root";
String password = "password";

try (Connection conn = DriverManager.getConnection(url, user, password)) {
    System.out.println("数据库连接成功");
} catch (SQLException e) {
    System.out.println("连接错误: " + e.getMessage());
}
```

```
jdbc:mysql://主机:端口/数据库名?参数
```

执行操作

```java
try (Connection conn = DriverManager.getConnection(url, user, password)) {
    Statement stmt = conn.createStatement();
    ResultSet rs = stmt.executeQuery("SELECT * FROM students");

    while (rs.next()) {
        System.out.println(rs.getString("name"));
    }
} catch (SQLException e) {
    System.out.println("数据库错误: " + e.getMessage());
}
```

## HTTP

HTTP 是一个基于请求-响应模式的无状态协议，意味着它不要求服务器保持请求之间的信息状态。客户端（通常是浏览器）发起一个请求到服务器，服务器回应请求的资源。这些资源可以是 HTML 文件、图片、视频片段或其他类型的数据。

### HTTP 请求

一个 HTTP 请求由以下几个部分组成：

- **方法**：定义操作类型的动词，如 GET（获取数据）、POST（提交数据）、PUT（更新资源）、DELETE（删除资源）等。
- **路径**：指定请求的资源的 URL 或 URI。
- **版本**：HTTP 的版本，如 HTTP/1.1。
- **头部字段**：包含请求的元数据，如用户代理、接受的内容类型、cookies 等。
- **请求体**（可选）：提交的数据体，通常与 POST 和 PUT 请求一起使用。

## Spring

Spring 技术栈是一系列基于 Spring 框架开发的项目和技术的集合，旨在提供全面的解决方案，覆盖从数据访问到消息传递、事务管理以及微服务架构等多个方面。下面是 Spring 技术栈中的一些关键组件：

1. Spring Framework
2. Spring Boot
3. Spring Data
4. Spring Security
5. ...

### Spring Framework

Spring Framework 是为了解决企业级开发中各种常见问题而创建的。它提供了很多功能，例如：

- **依赖注入（Dependency Injection）**：Spring 的核心功能，允许通过声明方式组装各种应用组件，而不是硬编码，这样可以增加程序的模块化和可测性。
- **面向切面编程（AOP）**：支持将方法的调用拦截到一个独立的切面中，从而分离系统服务与业务逻辑。
- **集成**：Spring 可以通过各种方式轻松集成其他技术，如 JPA、JTA、JDBC 等。

总之，Spring 框架提供了一套强大的工具和功能，可以大大简化企业级应用开发的复杂性，提高开发效率和代码质量。它已经成为 Java 开发领域最流行和广泛使用的框架之一。

```xml
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>6.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.11.3</version>
            <scope>test</scope>
        </dependency>
```

#### 控制反转（Inversion of Control）

控制反转是一种设计原则，用于改变程序中各个组件获取它们依赖对象的方式。传统的程序设计中，组件通常自行创建或直接查找其依赖的对象。而在 IoC 中，这种控制权被转移给了外部系统（通常是框架或容器），由外部系统来创建并管理这些依赖关系。

IoC 的主要好处是降低了组件间的耦合度。组件不需要知道如何创建它们的依赖项，也不需要知道依赖项的具体实现。这使得组件更容易被测试、维护和复用。

依赖注入是实现 IoC 的一种手段。通过依赖注入，对象的依赖（即它需要的其他对象）在运行时由另一个系统（通常是 IoC 容器）动态地注入到对象中。这可以通过构造器注入、属性注入或方法注入等多种方式实现。

#### Bean

在传统意义上，Java Bean 指的是遵循特定编码约定的 Java 类。

在 Spring 框架中，Bean 有更广泛的含义。指由 Spring IoC 容器管理的任何对象。Spring Beans 可以是数据访问对象、服务、控制器、配置类等。

如何注册一个 Bean？

```java
@Test
public void test() {
	GenericApplicationContext context = new GenericApplicationContext();

  context.registerBean(MyBean.class);
	context.refresh();

  MyBean myBean = context.getBean(MyBean.class);
  // ...
}
```

### MVC 设计模式

Model-View-Controller（MVC）广泛用于实现用户界面、数据和控制逻辑的有效分离。这种模式将应用程序分为三个核心组成部分，目的是分离内部业务逻辑和用户界面，从而使应用程序的开发、维护和扩展变得更加简单和清晰。

1. **模型（Model）**
   - **作用**：模型代表应用程序的数据逻辑。它直接管理数据、逻辑和规则，并且通常负责访问数据库或帮助管理应用程序的状态。
   - **特点**：模型是独立于用户界面的，它不关心数据会如何显示或被操作。
2. **视图（View）**
   - **作用**：视图是用户界面的部分，负责显示数据（即模型）给用户。视图会从模型获取数据，但并不直接进行业务逻辑的处理。
   - **特点**：视图只负责数据的展示，不处理数据本身。
3. **控制器（Controller）**
   - **作用**：控制器作为模型和视图之间的中介，处理用户的输入并调用模型和视图。它接收用户的输入命令，然后决定调用哪个模型的方法以及使用哪个视图来显示数据。
   - **特点**：控制器将用户的输入转化为对模型和视图的命令，确保数据流和控制逻辑的正确性。

### Spring Boot

Spring Boot 用于简化 Spring 应用的初始搭建以及开发过程。它是基于 Spring 框架的，用于简化配置和部署流程，使开发人员可以快速启动并运行 Spring 应用程序。

- **自动配置**：Spring Boot 能够自动配置项目，根据项目中添加的依赖关系，尽可能地自动配置 Spring 应用程序。例如，如果数据库依赖在项目的类路径上，Spring Boot 自动配置一个数据库连接池。
- **独立运行**：Spring Boot 应用程序包含了一个内嵌的服务器（默认是 Tomcat），这意味着你不需要单独安装 Web 服务器。你可以通过执行 `java -jar` 来运行你的应用程序。
- **无代码生成和 XML 配置**：Spring Boot 不需要生成代码或进行 XML 配置。它使用约定优于配置的原则，减少了必须编写的配置量。

#### 如何新建一个 Spring Boot 项目？

#### 如何定义一个接口

```java
@RestController
public class MyController{
  @GetMapping("path")
  public void getMessage(){
    return "Hello, World";
  }
}
```

#### 分层架构

在使用 Spring Framework 进行应用开发时，常常采用分层架构模式来组织代码，以提高代码的可维护性、可扩展性和解耦。典型的分层架构包括 Controller 层、Service 层和 Mapper 层（或 DAO 层），每一层都有其特定的职责。

#### 如何在 Spring Boot 中使用 JDBC

首先，在 application.yml 中配置数据库

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/demo
    username: root
    password: passw0rd
```

然后，使用 JdbcTemplate 来连接到数据库

```java
@Resource
JdbcTemplate jdbcTemplate;
```

#### 如何测试一个接口

https://apifox.com
