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

## 反射实现基础

#### 类加载器

类加载器（ClassLoader），顾名思义，即加载类的东西。在我们使用一个类之前，JVM需要先将该类的字节码文件（.class文件）从磁盘、网络或其他来源加载到内存中，并对字节码进行解析生成对应的Class对象，供程序随时使用。这就是类加载器的功能。
我们可以利用类加载器，实现类的动态加载。

#### 静态加载与动态加载

不管使用什么样的类加载器，类，都是在第一次被用到时，动态加载到JVM的。这句话有两层含义：

1).Java程序在运行时并不一定被完整加载，只有当发现该类还没有加载时，才去本地或远程查找类的.class文件并验证和加载；<br>
2).当程序创建了第一个对类的静态成员的引用（如类的静态变量、静态方法、构造方法——构造方法也是静态的）时，才会加载该类。<br>

Java在运行时调用类加载器加载.class文件的这个特性叫做：动态加载。

如果代码中通过new Foo()方式来创建对象，那么在编译时就会进行调用类加载器加载Foo.class文件，生成Foo的所对应的Class对象，这个特性叫做静态加载。

需要区分加载和初始化的区别，加载了一个类的.class文件，不意味着该Class对象被初始化，事实上，一个类的初始化包括3个步骤：

1).加载（Loading），由类加载器执行，查找字节码，并创建一个Class对象（只是创建）；<br>
2).链接（Linking），验证字节码，为静态域分配存储空间（只是分配，并不初始化该存储空间），解析该类创建所需要的对其它类的应用；<br>
3).初始化（Initialization），首先执行静态初始化块static{}，初始化静态变量，执行静态方法（如构造方法）。<br>

#### Class（类）

虚拟机在class文件的加载阶段，把类信息保存在方法区数据结构中，并在Java堆中生成一个Class对象，作为类信息的入口。

![img](/img/post-in/post-ref.jpg)

Class类用来描述java类的组成成分。简单说就是描述java类的模板。<br>
其中含有：

    描述java类的名称；
    描述java类的属性；
    描述java类来自哪个包；
    描述java类的父类；
    描述java类的构造方法；
    描述java类的普通方法

Class对象是java.lang.Class类的实例，Class没有公共构造方法，无法直接new CLass()创建，Class对象是在加载类时由Java虚拟机以及通过调用类加载器中的
defineClass方法自动构造的。

---

## 反射详细用法

上述介绍到Class类对象是类加载的时候创建，那么我们如何得到具体java类的Class对象呢？

下面以Code这个类为例：

对于普通的对象，我们一般都会这样创建和表示：

    Code code1 = new Code();

获得Code类的模板Class对象共有三种方式：

    Class c1 = Code.class;
    //这说明任何一个类都有一个隐含的静态成员变量class，这种方式是通过获取类的静态成员变量class得到的
    Class c2 = code1.getClass();
    //code1是Code的一个对象，这种方式是通过一个类的对象的getClass()方法获得的
    Class c3 = Class.forName("com.reflect.Code");
    //这种方法是Class类调用forName方法，通过一个类的全量限定名获得

注意，单个java类所对应的字节码模板只会有一份存在于虚拟机内存中，也就是说具体java类的Class对象只能有一个，上述我们通过三种方式获得了Code类的Class对象，
当此类的Class对象存在时，为直接获取，如果不存在时才会调用类加载器进行动态创建，这也是java的动态加载机制所体现。所以上面的三个对象c1，c2，c3都是指向同一个Class对象。

---

## 获取类的信息

我们知道了获取Class对象的三种方式，那么接下来看一看Class对象可以获取java类的哪些信息吧。

#### 类名

通过 getName() 方法返回类的全限定类名（包含包名）：

    String className = c3.getName();

如果你仅仅只是想获取类的名字(不包含包名)，那么你可以使用 getSimpleName()方法:

    String simpleClassName = c3.getSimpleName();

#### 修饰符

可以通过 Class 对象来访问一个类的修饰符， 即public,private,static 等等的关键字，你可以使用如下方法来获取类的修饰符：

    int modifiers = c3.getModifiers();

修饰符都被包装成一个int类型的数字，这样每个修饰符都是一个位标识(flag bit)，这个位标识可以设置和清除修饰符的类型。 可以使用 java.lang.reflect.Modifier
类中的方法来检查修饰符的类型：

    Modifier.isAbstract(int modifiers);
    Modifier.isFinal(int modifiers);
    Modifier.isInterface(int modifiers);
    Modifier.isNative(int modifiers);
    Modifier.isPrivate(int modifiers);
    Modifier.isProtected(int modifiers);
    Modifier.isPublic(int modifiers);
    Modifier.isStatic(int modifiers);
    Modifier.isStrict(int modifiers);
    Modifier.isSynchronized(int modifiers);
    Modifier.isTransient(int modifiers);
    Modifier.isVolatile(int modifiers);

#### 包信息

可以使用 Class 对象通过如下的方式获取包信息：

    Package package = c3.getPackage();

#### 父类

通过 Class 对象你可以访问类的父类：

    Class superclass = c3.getSuperclass();

可以看到 superclass 对象其实就是一个 Class 类的实例，所以你可以继续在这个对象上进行反射操作。

#### 接口

可以通过如下方式获取指定类所实现的接口集合：

    Class[] interfaces = c3.getInterfaces();

由于一个类可以实现多个接口，因此 getInterfaces(); 方法返回一个 Class 数组，在 Java 中接口同样有对应的 Class 对象。 注意：getInterfaces() 方法仅仅只返回当前类所实现的接口。当前类的父类如果实现了接口，这些接口是不会在返回的 Class 集合中的，尽管实际上当前类其实已经实现了父类接口。

#### 构造器

我们可以通过 getConstructors() 方法来获取 Constructor 类的实例：

    Constructor[] constructors = c3.getConstructors();

返回的 Constructor 数组包含每一个声明为公有的（Public）构造方法。 如果你知道你要访问的构造方法的方法参数类型，你可以用下面的方法获取指定的构造方法，这例子返回的构造方法的方法参数为 String 类型：

    Constructor constructor = c3.getConstructor(new Class[]{String.class});

如果没有指定的构造方法能满足匹配的方法参数则会抛出：NoSuchMethodException。

构造方法参数你可以通过如下方式获取指定构造方法的方法参数信息：

    Class[] parameterTypes = constructor.getParameterTypes();

利用 Constructor 对象实例化一个类，你可以通过如下方法实例化一个类：

    Constructor constructor = MyObject.class.getConstructor(String.class);
    MyObject myObject = (MyObject)
    constructor.newInstance("constructor-arg1");

constructor.newInstance()方法的方法参数是一个可变参数列表，但是当你调用构造方法的时候你必须提供精确的参数，即形参与实参必须一一对应。在这个例子中构造方法需要一个 String 类型的参数，那我们在调用 newInstance 方法的时候就必须传入一个 String 类型的参数。

#### 方法

可以通过 getMethods() 方法获取 Method 对象：

    Method[] methods = c3.getMethods();

返回的 Method 对象数组包含了指定类中声明为公有的(public)的所有变量集合。

如果你知道你要调用方法的具体参数类型，你就可以直接通过参数类型来获取指定的方法，下面这个例子中返回方法对象名称是“doSomething”，他的方法参数是 String 类型：

    Class  c3 = ...//获取Class对象
    Method method = c3.getMethod("doSomething", new Class[]{String.class});

如果根据给定的方法名称以及参数类型无法匹配到相应的方法，则会抛出 NoSuchMethodException。 如果你想要获取的方法没有参数，那么在调用 getMethod()方法时第二个参数传入 null 即可，就像这样：

    Class  c3 = ...//获取Class对象
    Method method = c3.getMethod("doSomething", null);

方法参数以及返回类型
你可以获取指定方法的方法参数是哪些：

    Method method = ... //获取Class对象
    Class[] parameterTypes = method.getParameterTypes();

你可以获取指定方法的返回类型：

    Method method = ... //获取Class对象
    Class returnType = method.getReturnType();

通过 Method 对象调用方法
你可以通过如下方式来调用一个方法：

    //获取一个方法名为doSomesthing，参数类型为String的方法
    Method method = MyObject.class.getMethod("doSomething", String.class);
    Object returnValue = method.invoke(null, "parameter-value1");

传入的 null 参数是你要调用方法的对象，如果是一个静态方法调用的话则可以用 null 代替指定对象作为 invoke()的参数，在上面这个例子中，如果 doSomething 不是静态方法的话，你就要传入有效的 MyObject 实例而不是 null。 Method.invoke(Object target, Object … parameters)方法的第二个参数是一个可变参数列表，但是你必须要传入与你要调用方法的形参一一对应的实参。就像上个例子那样，方法需要 String 类型的参数，那我们必须要传入一个字符串。

#### 变量

你可以通过如下方式访问一个类的成员变量：

    Field[] method = c3.getFields();

在通常的观点中从对象的外部访问私有变量以及方法是不允许的，但是 Java 反射机制可以做到这一点。使用这个功能并不困难，在进行单元测试时这个功能非常有效。本节会向你展示如何使用这个功能。

注意：这个功能只有在代码运行在单机 Java 应用(standalone Java application)中才会有效,就像你做单元测试或者一些常规的应用程序一样。如果你在 Java Applet 中使用这个功能，那么你就要想办法去应付 SecurityManager 对你限制了。但是一般情况下我们是不会这么做的，所以在本节里面我们不会探讨这个问题。

访问私有变量
要想获取私有变量你可以调用 Class.getDeclaredField(String name)方法或者 Class.getDeclaredFields()方法。

    Class.getField(String name)和 Class.getFields()只会返回公有的变量，无法获取私有变量。下面例子定义了一个包含私有变量的类，在它下面是如何通过反射获取私有变量的例子：

    public class PrivateObject {

      private String privateString = null;

      public PrivateObject(String privateString) {
        this.privateString = privateString;
      }
    }
    PrivateObject privateObject = new PrivateObject("The Private Value");

    Field privateStringField = PrivateObject.class.
                getDeclaredField("privateString");

    privateStringField.setAccessible(true);

    String fieldValue = (String) privateStringField.get(privateObject);
    System.out.println("fieldValue = " + fieldValue);

这个例子会输出”fieldValue = The Private Value”，The Private Value 是 PrivateObject 实例的 privateString 私有变量的值，注意调用 PrivateObject.class.getDeclaredField(“privateString”)方法会返回一个私有变量，这个方法返回的变量是定义在 PrivateObject 类中的而不是在它的父类中定义的变量。 注意 privateStringField.setAccessible(true)这行代码，通过调用 setAccessible()方法会关闭指定类 Field 实例的反射访问检查，这行代码执行之后不论是私有的、受保护的以及包访问的作用域，你都可以在任何地方访问，即使你不在他的访问权限作用域之内。但是你如果你用一般代码来访问这些不在你权限作用域之内的代码依然是不可以的，在编译的时候就会报错。

访问私有方法
访问一个私有方法你需要调用 Class.getDeclaredMethod(String name, Class[] parameterTypes)或者 Class.getDeclaredMethods() 方法。 Class.getMethod(String name, Class[] parameterTypes)和 Class.getMethods()方法，只会返回公有的方法，无法获取私有方法。下面例子定义了一个包含私有方法的类，在它下面是如何通过反射获取私有方法的例子：

    public class PrivateObject {

      private String privateString = null;

      public PrivateObject(String privateString) {
        this.privateString = privateString;
      }

      private String getPrivateString(){
        return this.privateString;
      }
    }
    PrivateObject privateObject = new PrivateObject("The Private Value");

    Method privateStringMethod = PrivateObject.class.
            getDeclaredMethod("getPrivateString", null);

    privateStringMethod.setAccessible(true);

    String returnValue = (String)
            privateStringMethod.invoke(privateObject, null);

    System.out.println("returnValue = " + returnValue);

这个例子会输出”returnValue = The Private Value”，The Private Value 是 PrivateObject 实例的 getPrivateString()方法的返回值。 PrivateObject.class.getDeclaredMethod(“privateString”)方法会返回一个私有方法，这个方法是定义在 PrivateObject 类中的而不是在它的父类中定义的。 同样的，注意 Method.setAcessible(true)这行代码，通过调用 setAccessible()方法会关闭指定类的 Method 实例的反射访问检查，这行代码执行之后不论是私有的、受保护的以及包访问的作用域，你都可以在任何地方访问，即使你不在他的访问权限作用域之内。但是你如果你用一般代码来访问这些不在你权限作用域之内的代码依然是不可以的，在编译的时候就会报错。

#### 泛型

运用泛型反射的经验法则

下面是两个典型的使用泛型的场景：

1、声明一个需要被参数化（parameterizable）的类/接口。

2、使用一个参数化类。

当你声明一个类或者接口的时候你可以指明这个类或接口可以被参数化， java.util.List 接口就是典型的例子。你可以运用泛型机制创建一个标明存储的是 String 类型 list，这样比你创建一个 Object 的l ist 要更好。
当你想在运行期参数化类型本身，比如你想检查 java.util.List 类的参数化类型，你是没有办法能知道他具体的参数化类型是什么。这样一来这个类型就可以是一个应用中所有的类型。但是，当你检查一个使用了被参数化的类型的变量或者方法，你可以获得这个被参数化类型的具体参数。
总之：
你不能在运行期获知一个被参数化的类型的具体参数类型是什么，但是你可以在用到这个被参数化类型的方法以及变量中找到他们，换句话说就是获知他们具体的参数化类型。在下面的段落中会向你演示这类情况。

泛型方法返回类型
如果你获得了 java.lang.reflect.Method 对象，那么你就可以获取到这个方法的泛型返回类型信息。如果方法是在一个被参数化类型之中（译者注：如 T fun()）那么你无法获取他的具体类型，但是如果方法返回一个泛型类（译者注：如 List fun()）那么你就可以获得这个泛型类的具体参数化类型。你可以在“Java Reflection: Methods”中阅读到有关如何获取Method对象的相关内容。下面这个例子定义了一个类这个类中的方法返回类型是一个泛型类型：

    public class MyClass {

        protected List<String> stringList = ...;

        public List<String> getStringList(){
            return this.stringList;
        }
    }

我们可以获取 getStringList()方法的泛型返回类型，换句话说，我们可以检测到 getStringList()方法返回的是 List 而不仅仅只是一个 List。如下例：

    Method method = MyClass.class.getMethod("getStringList", null);

    Type returnType = method.getGenericReturnType();

    if(returnType instanceof ParameterizedType){
        ParameterizedType type = (ParameterizedType) returnType;
        Type[] typeArguments = type.getActualTypeArguments();
        for(Type typeArgument : typeArguments){
            Class typeArgClass = (Class) typeArgument;
            System.out.println("typeArgClass = " + typeArgClass);
        }
    }

这段代码会打印出 “typeArgClass = java.lang.String”，Type[]数组typeArguments 只有一个结果 – 一个代表 java.lang.String 的 Class 类的实例。Class 类实现了 Type 接口。

泛型方法参数类型

你同样可以通过反射来获取方法参数的泛型类型，下面这个例子定义了一个类，这个类中的方法的参数是一个被参数化的 List：

    public class MyClass {
      protected List<String> stringList = ...;

      public void setStringList(List<String> list){
        this.stringList = list;
      }
    }

你可以像这样来获取方法的泛型参数：

    method = Myclass.class.getMethod("setStringList", List.class);

    Type[] genericParameterTypes = method.getGenericParameterTypes();

    for(Type genericParameterType : genericParameterTypes){
        if(genericParameterType instanceof ParameterizedType){
            ParameterizedType aType = (ParameterizedType) genericParameterType;
            Type[] parameterArgTypes = aType.getActualTypeArguments();
            for(Type parameterArgType : parameterArgTypes){
                Class parameterArgClass = (Class) parameterArgType;
                System.out.println("parameterArgClass = " + parameterArgClass);
            }
        }
    }

这段代码会打印出”parameterArgType = java.lang.String”。Type[]数组 parameterArgTypes 只有一个结果 – 一个代表 java.lang.String 的 Class 类的实例。Class 类实现了Type接口。

泛型变量类型
同样可以通过反射来访问公有（Public）变量的泛型类型，无论这个变量是一个类的静态成员变量或是实例成员变量。你可以在“Java Reflection: Fields”中阅读到有关如何获取 Field 对象的相关内容。这是之前的一个例子，一个定义了一个名为 stringList 的成员变量的类。

    method = Myclass.class.getMethod("setStringList", List.class);

    Type[] genericParameterTypes = method.getGenericParameterTypes();

    for(Type genericParameterType : genericParameterTypes){
        if(genericParameterType instanceof ParameterizedType){
            ParameterizedType aType = (ParameterizedType) genericParameterType;
            Type[] parameterArgTypes = aType.getActualTypeArguments();
            for(Type parameterArgType : parameterArgTypes){
                Class parameterArgClass = (Class) parameterArgType;
                System.out.println("parameterArgClass = " + parameterArgClass);
            }
        }
    }

这段代码会打印出”fieldArgClass = java.lang.String”。Type[]数组 fieldArgClass 只有一个结果 – 一个代表 java.lang.String 的 Class 类的实例。Class 类实现了 Type 接口。

---

## 反射应用实例

---

## 反射的性能问题

---

## 反射总结

反射就是把java类中的组成部分映射成相应的java对象、字节模板

---

## 引用链接

[http://www.jianshu.com/p/2315dda64ad2](http://www.jianshu.com/p/2315dda64ad2)<br>
[http://www.cnblogs.com/jqyp/archive/2012/03/29/2423112.html](http://www.cnblogs.com/jqyp/archive/2012/03/29/2423112.html)<br>
[http://blog.csdn.net/trigl/article/details/51042403](http://blog.csdn.net/trigl/article/details/51042403)<br>
[http://www.cnblogs.com/rollenholt/archive/2011/09/02/2163758.html](http://www.cnblogs.com/rollenholt/archive/2011/09/02/2163758.html)<br>
[http://www.cnblogs.com/hxsyl/archive/2013/03/23/2977593.html](http://www.cnblogs.com/hxsyl/archive/2013/03/23/2977593.html)<br>
