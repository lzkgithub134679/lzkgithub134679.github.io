---
title: java 自定义注解的学习和使用
date: 2019-06-28 13:40:12
tags:
- Show All
- annotation
header-img: "Demo.png"
---

### 一、注解说明

Java注解又称Java标注，是Java语言5.0版本开始支持加入源代码的特殊语法元数据。为我们在代码中添加信息提供了一种形式化的方法，使我们可以在稍后某个时刻非常方便的使用这些数据。
Java语言中的类、方法、变量、参数和包等都可以被标注。和Javadoc不同，Java标注可以通过反射获取注解内容。在编译器生成类文件时，注解可以被嵌入到字节码中。Java虚拟机可以保留注解内容，在运行时可以获取到注解内容。

### 二、注解介绍

1、JDK1.5版本内置了三种标准注解:

- @Override：表示当前的方法定义将覆盖父类中的方法。
- @Deprecated ：表示被弃用的代码。
- @SuppressWarnings：关闭对当前的代码的警告。

2、在java.lang.annotation：中还有四种注解。通常经常自己定义注解使用，都会用到。

- @Retention：表示需要在什么级别保存该注解信息
- @Documented：将注解包含在Javadoc中
- @Target：表示该注解可以用于什么地方
- @Inherited：允许子类继承父类中的注解

@Retention可选参数有：

`SOURCE`：注解将被编译器丢弃
`CLASS`：注解在class文件中可用，但会被VM丢弃
`RUNTIME`：VM将在运行期间保留注解，因此可以通过反射机制读取注解的信息

@Targer：可选参数有：

`CONSTRUCTOR`：构造器的声明
`FIELD`：域声明（包括`enum`实例）
`LOCAL_VARIABLE`：局部变量声明
`METHOD`：方法声明
`PACKAGE`：包声明
`PARAMETER`：参数声明
`TYPE`：类、接口（包括注解类型）或`enum`声明