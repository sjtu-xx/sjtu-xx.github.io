---
title: JAVA编程思想
date: 2021-06-15 21:44:57
tags:
categories:
- JAVA
- 提升
---

《on Java 8》是Bruce Eckel继《Java编程思想(第四版)》（Thinking in Java）后的续作，主要是在《Java编程思想》的基础上，增加了对Java8新特性的一些理解。

由于这本书面向所有读者，本笔记不再对Java基础部分进行记录，只记录一些在Java编程过程中经常会遇到的一些问题。

<!--more-->

## 对象

>  Java中的参数传递：Java 对象标识符实际上是“对象引用”（object references），并且一切都是值传递。所以你不是通过引用传递，而是“通过值传递对象引用。

### Java中基本数据类型

#### 基本数据类型所占的空间

| 基本类型 | 大小    | 最小值    | 最大值         | 包装类型  |
| -------- | ------- | --------- | -------------- | --------- |
| boolean  | —       | —         | —              | Boolean   |
| char     | 16 bits | Unicode 0 | Unicode 216 -1 | Character |
| byte     | 8 bits  | -128      | +127           | Byte      |
| short    | 16 bits | - 215     | + 215 -1       | Short     |
| int      | 32 bits | - 231     | + 231 -1       | Integer   |
| long     | 64 bits | - 263     | + 263 -1       | Long      |
| float    | 32 bits | IEEE754   | IEEE754        | Float     |
| double   | 64 bits | IEEE754   | IEEE754        | Double    |
| void     | —       | —         | —              | Void      |

所有的数值类型都是有正/负符号的。布尔（boolean）类型的大小没有明确的规定，通常定义为取字面值 “true” 或 “false” 。基本类型有自己对应的包装类型，如果你希望在堆内存里表示基本类型的数据，就需要用到它们的包装类。

#### 基本数据类型的初始化

| 基本类型 | 初始值        |
| -------- | ------------- |
| boolean  | false         |
| char     | \u0000 (null) |
| byte     | (byte) 0      |
| short    | (short) 0     |
| int      | 0             |
| long     | 0L            |
| float    | 0.0f          |
| double   | 0.0d          |

> 为了安全，我们最好始终显式地初始化变量（类中的变量）。

