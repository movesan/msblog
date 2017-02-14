---
layout:     post
title:      "Java核心技术---反射（reflecttion）"
subtitle:   " Reflection-oriented "
date:       2017-02-13 22:00:00
author:     "Movesan"
header-img: "img/post-bg/post-bg-java.jpg"
catalog: true
tags:
    - Java

---

## 引言

我们在学习java的过程中都会遇到一个技术点---反射（reflection），起初可能是作为了解，但随着学习的时间越来越久，接触的框架越来越多，就会意识到这是个不可绕过的知识点。
就像我们游戏打副本一样，BOSS不能略过，因为奖励多多。所以话不多说，一起攻克它吧！

---

## 什么是反射

反射：在计算机科学中，反射是指计算机程序在运行时（Run time）可以访问、检测和修改它本身状态或行为的一种能力。用比喻来说，那种程序能够“观察”并且修改自己的行为。
这是维基百科中对于反射的概念描述。

java反射：java反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

反射是java语言的一个特性，它允许程序在运行时（注意不是编译的时候）来进行自我检查并且对内部的成员进行操作。例如它允许一个java的类获取他所有的成员变量和方法并且显示出来。Java 的这一能力在实际应用中也许用得不是很多，
但是在其它的程序设计语言中根本就不存在这一特性。例如，Pascal、C 或者 C++ 中就没有办法在程序中获得函数定义相关的信息。 （来自Sun）

简单的来说，反射机制指的是程序在运行时能够获取自身的信息。在java中，只要给定类的名字，那么就可以通过反射机制来获得类的所有信息。

---

## 初识反射机制

有些时候，我们用过一些知识，但是并不知道它的专业术语是什么，在刚刚学jdbc时用过一行代码，Class.forName("com.mysql.jdbc.Driver.class").newInstance();但是那时候只知道那行代码是生成
驱动对象实例，并不知道它的具体含义。听了反射机制这节课后，才知道，原来这就是反射，现在很多开源框架都用到反射机制，springMVC、hibernate、struts都是用反射机制实现的。

---

## 为什么要用反射



---

## 反射基础

#### 类加载器

类加载器（ClassLoader），顾名思义，即加载类的东西。在我们使用一个类之前，JVM需要先将该类的字节码文件（.class文件）从磁盘、网络或其他来源加载到内存中，并对字节码进行解析生成对应的Class对象，这就是类加载器的功能。
我们可以利用类加载器，实现类的动态加载。

#### 静态加载与动态加载

不管使用什么样的类加载器，类，都是在第一次被用到时，动态加载到JVM的。这句话有两层含义：<br>
1.Java程序在运行时并不一定被完整加载，只有当发现该类还没有加载时，才去本地或远程查找类的.class文件并验证和加载；<br>
2.当程序创建了第一个对类的静态成员的引用（如类的静态变量、静态方法、构造方法——构造方法也是静态的）时，才会加载该类。<br>

Java在运行时调用类加载器加载.class文件的这个特性叫做：动态加载。

如果代码中通过new Foo()方式来创建对象，那么在编译时就会进行调用类加载器加载Foo.class文件，生成Foo的所对应的Class对象，这个特性叫做静态加载。

需要区分加载和初始化的区别，加载了一个类的.class文件，不意味着该Class对象被初始化，事实上，一个类的初始化包括3个步骤：<br>
1).加载（Loading），由类加载器执行，查找字节码，并创建一个Class对象（只是创建）；<br>
2).链接（Linking），验证字节码，为静态域分配存储空间（只是分配，并不初始化该存储空间），解析该类创建所需要的对其它类的应用；<br>
3).初始化（Initialization），首先执行静态初始化块static{}，初始化静态变量，执行静态方法（如构造方法）。<br>

#### Class（类）

虚拟机在class文件的加载阶段，把类信息保存在方法区数据结构中，并在Java堆中生成一个Class对象，作为类信息的入口。

![img](/img/post-in/post-ref.jpg)

其是java.lang.Class类的实例对象，Class没有公共构造方法，无法直接new CLass()创建，Class对象是在加载类时由Java虚拟机以及通过调用类加载器中的defineClass方法自动构造的。

---

## 反射详细用法

---

## 反射应用实例

---

## 反射的性能问题

---

## 引用链接

http://www.cnblogs.com/paddix  java技术

http://www.jianshu.com/p/2315dda64ad2,
http://www.jianshu.com/p/808a36134da5,
http://www.cnblogs.com/zhguang/p/3154584.html

[http://www.cnblogs.com/jqyp/archive/2012/03/29/2423112.html](http://www.cnblogs.com/jqyp/archive/2012/03/29/2423112.html)<br>
[http://blog.csdn.net/trigl/article/details/51042403](http://blog.csdn.net/trigl/article/details/51042403)<br>
[http://www.cnblogs.com/rollenholt/archive/2011/09/02/2163758.html](http://www.cnblogs.com/rollenholt/archive/2011/09/02/2163758.html)<br>
[http://www.cnblogs.com/hxsyl/archive/2013/03/23/2977593.html](http://www.cnblogs.com/hxsyl/archive/2013/03/23/2977593.html)<br>
