---
layout:     post
title:      "Java static 关键字及其用法 "
subtitle:   " 使用 static 声明静态代码块可以优化程序性能 "
date:       2019-04-17 08:50:00
author:     "Edger"
header-img: "img/post-bg-Java-static.jpg"
catalog:    true
tags:

    - Java
    
    - static
---


> Java static 关键字是在编写代码或者阅读代码是经常用到或遇到的一个关键字，因此我们有必要理解并掌握 static 的用法。

### 1. static 是什么？

`static` 是静态修饰符。什么是静态修饰符？在程序中，任何变量或者代码都是在编译时由系统自动分配内存来存储的。而所谓静态，就是指在编译后系统所分配的内存会一直存在，直到程序退出内存才会释放这个空间。也就是说，只要程序在运行，那么这块内存就会一直存在。

这样做有什么意义呢？在 Java 程序里面，所有的东西都是对象，而对象的抽象就是类。对于一个类而言，如果要使用它的成员，那么普通情况下必须先实例化对象后，通过对象的引用才能够访问这些成员，但是用 static 修饰的成员可以通过 “`类名 . 成员`” 的方式进行直接访问（这里不考虑类的访问控制）。

static 表示 “**全局**” 或者 “**静态**” 的意思，用来修饰成员变量和成员方法，也可以用来形成静态 static 代码块。但是需要明白的是， Java 语言中是没有全局变量这个概念的。

被 static 修饰的成员变量和成员方法独立于该类的任何对象，它们属于类而不是类的实例，被类的所有实例共享。

只要这个类被加载，JVM 就能根据类名在运行时数据区的方法区内定找到它们。因此，static 对象可以在它的任何对象创建之前访问，无需引用任何对象。

### 2. 为什么要设计 static ？

> static 被设计出来的原因（好处）：

- 类直接调用成员。使用 static 修饰的成员可以通过 “类名 . 成员” 的方式直接调用。 

- 优先加载，兼顾优化性能。使用 static 修饰的变量，在内存中只有一个拷贝，可以节省内存；static 可以用来声明静态代码块，静态代码块只会执行一次。JVM 在加载类时会执行静态代码块，所以静态代码块先于主方法执行。如果类中包含多个静态代码块，会按照顺序执行。静态代码块可以优化程序性能。为什么这样说？详见 3.3 小节。

### 3. static 的用法

#### 3.1 静态变量

```java
static String str = "This is a static variable.";
```
被 static 修饰的变量被称作静态变量，静态变量和非静态变量的区别是：

- 静态变量被所有的对象所共享，在内存中只有一个副本，它当且仅当在类初次加载时会被初始化。
    
- 非静态变量是对象所拥有的，在创建对象的时候被初始化，存在多个副本，各个对象拥有的副本互不影响。

static 成员变量的初始化顺序按照定义的顺序进行初始化。

#### 3.2 静态方法

```java
    public static void printString() {
        System.out.println("This is a static method.");
    }
```

静态方法可以直接通过类名调用，任何的实例也都可以调用 .

static 方法独立于任何实例，因此 static 方法必须被实现，而不能是抽象的 abstract 。

例如为了方便方法的调用，Java API 中的 Math 类中所有的方法都是静态的，而一般类内部的 static 方法也是方便其它类对该方法的调用。

静态方法是类内部的一类特殊方法，只有在需要时才将对应的方法声明成静态的，一个类内部的方法一般都是非静态的 .

因此，如果需要在不创建对象的情况下调用某个方法，则可以将这个方法声明为 static 。

最常见的 static 方法就是 main 方法，至于为什么 main 方法必须是 static 的，现在就很清楚了。因为程序在执行 main 方法的时候还没有创建任何对象，因此只有通过类名来访问。

另外记住，即使没有显示地声明为 static ，类的构造器实际上也是静态方法。

#### 3.3 静态代码块

static 代码块也叫静态代码块，是在类中独立于类成员的 static 语句块。一个类中，静态代码块可以有多个，位置随意，它不在任何的方法体内。JVM 加载类时会执行这些静态的代码块，如果有多个 static 代码块，JVM 将按照它们在类中出现的先后顺序依次执行它们，**每个代码块只会被执行一次**。

为什么说静态代码块可以优化程序性能？因为它的特性：**只会在类加载的时候执行一次。**

下面举例说明：

```java
class Person {
    private Date birthDate;

    public Person(Date birthDate) {
        this.birthDate = birthDate;
    }

    boolean isBornBetween1946And1964() {
        Date startDate = Date.valueOf("1946");
        Date endDate = Date.valueOf("1964");
        return birthDate.compareTo(startDate) >= 0 && birthDate.compareTo(endDate) < 0;
    }
}
```

Person 是一个非常简单的类，其中 `isBornBetween1946And1964( )` 是用来这个人是否是` 1946～1964` 年出生的，而每次 `isBornBetween1946And1964( )` 被调用的时候，都会生成 `startDate` 和 `birthDate` 两个对象，造成了空间浪费，如果改成这样效率会更好：

```java
class Person {
    private Date birthDate;
    private static Date startDate, endDate;
    
    static {
        startDate = Date.valueOf("1946");
        endDate = Date.valueOf("1964");
    }

    public Person(Date birthDate) {
        this.birthDate = birthDate;
    }

    boolean isBornBetween1946And1964() {
        return birthDate.compareTo(startDate) >= 0 && birthDate.compareTo(endDate) < 0;
    }
}
```

所以，很多时候，可以将一些只需要执行一次的初始化操作放在 static 代码块中执行，有利于提高程序的效率。

### 4. final static 的作用

```java
private static final String TAG = "TAG";
```

`static final` 用来修饰成员变量和成员方法，可简单理解为 Java 中的 “全局常量”。

对于变量，使用 `static final` 修饰后，表示一旦给值就不可修改，并且通过类名可以访问。

对于方法，使用 `static final` 修饰后，则表示不可覆盖，并且可以通过类名直接访问。

对于被 static 和 final 修饰过的实例常量，实例本身不能再改变了。但对于一些容器类型（比如，ArrayList、HashMap）的实例变量，不可以改变的只是容器变量本身，但可以修改容器中存放的对象，这一点在编程中用到很多。

### 5. static 的注意事项

#### 5.1 static 关键字会改变类中成员的访问权限吗？

有些初学的朋友会将 Java 中的 static 与 C/C++ 中的 static 关键字的功能混淆了。在这里只需要记住一点：与 C/C++ 中的 static 不同，Java 中的 static 关键字不会影响到变量或者方法的作用域（ C++ 中的 static 则会隐藏变量，让其他 C++ 文件不能访问）。

在 Java 中能够影响到访问权限的只有 private、public、protected（包括包访问权限）这几个关键字。看下面的例子就明白了：

![](/img/in-post/post-Java-static-access-right.png)

打印 Person.age 时提示错误 "Person.age is not visible."，但是打印 Person.name 则不会，这说明 static 关键字并不会改变变量和方法的访问权限。

#### 5.2 能通过 this 访问静态成员变量吗？

在静态方法里只能直接调用同类中其他的静态成员（包括变量和方法），而不能直接访问类中的非静态成员。这是因为，对于非静态的方法和变量，需要先创建类的实例对象后才可使用，而静态方法在使用前不用创建任何对象。

静态方法不能以任何方式引用 this 和 super 关键字，因为静态方法在使用前不用创建任何实例对象，当静态方法调用时，this 所引用的对象根本没有产生。

#### 5.3 static 能作用于局部变量么？

在 C/C++ 中 static 是可以作用域局部变量的，但是在 Java 中切记：static 是不允许用来修饰局部变量。不要问为什么，这是 Java 语法的规定。

具体原因可以参考这篇博文的讨论：[Java static 关键字为什么不能应用于局部变量？](http://www.debugease.com/j2se/178932.html)

### 6. 总结

> 静态变量

通常情况下，一个类的成员必须通过该类的对象访问。但是使用 static 修饰类的成员后，它就能够被类本身使用，而不必引用该类的实例（对象）。

声明为 static 的变量实质上就是全局变量。当声明一个类的对象时，并不会产生 static 变量的拷贝，而是该类所有的实例共用同一个 static 变量。

> 静态方法

静态方法最常见的例子是` main( )`。因为在程序开始执行时必须调用` main( )`，所以它需要被声明为 static 。

声明为 static 的方法有以下几条限制：

- 仅能调用其他的 static 方法。

- 只能访问 static 数据。

- 不能以任何方式引用 this 或 super。

> 静态代码块

如果需要通过计算来初始化 static 变量，可以声明一个 static 块，static 块仅在该类被加载时执行一次。

---

本文参考文章如下：

- [Java 为什么要设计静态方法？这样设计的目的是什么 ?](https://www.zhihu.com/question/51258904)
- [Java static 关键字理解](https://www.jianshu.com/p/b1259f641d09)
- [Java 修饰符 static 的作用](https://blog.csdn.net/yuxin1100/article/details/51679303)
- [static 静态修饰符](https://www.jianshu.com/p/289d3c1735f0)
- [static 修饰符详解](https://blog.csdn.net/u012152619/article/details/46003303)
- [Java 中的 static 关键字解析](https://www.cnblogs.com/dolphin0520/p/3799052.html)
- [Java 中 static 作用及用法详解](https://blog.csdn.net/fengyuzhengfan/article/details/38082999)