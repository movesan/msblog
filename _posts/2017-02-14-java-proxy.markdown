---
layout:     post
title:      "Java核心技术---代理（proxy）"
subtitle:   " Proxy For Java "
date:       2017-02-14 22:00:00
author:     "Movesan"
header-img: "img/post-bg/post-bg-java.jpg"
catalog: true
tags:
    - Java

---

## 引言

代理模式是常用的设计模式，其特征是代理类与委托类具有相同的接口，在具体实现上，有静态代理和动态代理之分。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，
代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务，也就是说代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。

---

## 什么是代理

要理解代理模式很简单，其实生活当中就存在代理模式：

我们购买火车票可以去火车站买，但是也可以去火车票代售处买，此处的火车票代售处就是火车站购票的代理，即我们在代售点发出买票请求，代售点会把请求发给火车站，火车站把购买成功响应发给代售点，
代售点再告诉你。但是代售点只能买票，不能退票，而火车站能买票也能退票，因此代理对象支持的操作可能和委托对象的操作有所不同。

再举一个写程序会碰到的一个例子：

如果现在有一个已有项目（你没有源代码，只能调用它）能够调用 int compute(String exp1) 实现对于后缀表达式的计算，你想使用这个项目实现对于中缀表达式的计算，那么你可以写一个代理类，
并且其中也定义一个compute(String exp2)，这个exp2参数是中缀表达式，因此你需要在调用已有项目的 compute() 之前将中缀表达式转换成后缀表达式(Preprocess)，再调用已有项目的compute()，
当然你还可以接收到返回值之后再做些其他操作比如存入文件(Postprocess)，这个过程就是使用了代理模式。

在平时用电脑也会碰到代理模式的应用：

远程代理：我们在国内因为GFW，所以不能访问 facebook，我们可以用翻墙（设置代理）的方法访问。访问过程是：<br>
1)用户把HTTP请求发给代理<br>
2)代理把HTTP请求发给web服务器<br>
3)web服务器把HTTP响应发给代理<br>
4)代理把HTTP响应发回给用户<br>

---

## 代理模式

定义：给某个对象提供一个代理对象，并由代理对象控制对于原对象的访问，即客户不直接操控原对象，而是通过代理对象间接地操控原对象。

![img](/img/post-in/post-proxy.png)

在上图中：

*   RealSubject 是原对象（本文把原对象称为"委托对象"），Proxy 是代理对象。
*   Subject 是委托对象和代理对象都共同实现的接口。
*   Request() 是委托对象和代理对象共同拥有的方法。

代理的实现分为：

静态代理：代理类是在编译时就实现好的。也就是说 Java 编译完成后代理类是一个实际的 class 文件。<br>
动态代理：代理类是在运行时生成的。也就是说 Java 编译完之后并没有实际的 class 文件，而是在运行时动态生成的类字节码，并加载到JVM中。

下面以一个模拟需求说明静态代理和动态代理：委托类要处理一项耗时较长的任务，客户类需要打印出执行任务消耗的时间。
解决这个问题需要记录任务执行前时间和任务执行后时间，两个时间差就是任务执行消耗的时间。

---

## 静态代理

所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。

代理接口：

    /**  
     * 代理接口。处理给定名字的任务。
     */  
    public interface Subject {  
      /**
       * 执行给定名字的任务。
        * @param taskName 任务名
       */  
       public void dealTask(String taskName);   
    }  

委托类，具体处理业务：

    /**
     * 真正执行任务的类，实现了代理接口。
     */  
    public class RealSubject implements Subject {  

     /**
      * 执行给定名字的任务。这里打印出任务名，并休眠500ms模拟任务执行了很长时间
      * @param taskName  
      */  
       @Override  
       public void dealTask(String taskName) {  
          System.out.println("正在执行任务："+taskName);  
          try {  
             Thread.sleep(500);  
          } catch (InterruptedException e) {  
             e.printStackTrace();  
          }  
       }  
    }

静态代理类：

    /**
     *　代理类，实现了代理接口。
     */  
    public class ProxySubject implements Subject {  
        //代理类持有一个委托类的对象引用  
        private Subject delegate;  

        public ProxySubject(Subject delegate) {  
          this.delegate = delegate;  
        }  

       /**
        * 将请求分派给委托类执行，记录任务执行前后的时间，时间差即为任务的处理时间
        *  
        * @param taskName
        */  
       @Override  
       public void dealTask(String taskName) {  
         long stime = System.currentTimeMillis();   
         //将请求分派给委托类处理  
         delegate.dealTask(taskName);  
         long ftime = System.currentTimeMillis();   
         System.out.println("执行任务耗时"+(ftime - stime)+"毫秒");  

       }  
    }  

生成静态代理类工厂：

    public class SubjectStaticFactory {  
       //客户类调用此工厂方法获得代理对象。  
       //对客户类来说，其并不知道返回的是代理类对象还是委托类对象。  
       public static Subject getInstance(){   
         return new ProxySubject(new RealSubject());  
       }  
    }  

客户类:

    public class Client1 {  

       public static void main(String[] args) {  
          Subject proxy = SubjectStaticFactory.getInstance();  
          proxy.dealTask("DBQueryTask");  
       }   

    }

静态代理类优缺点

优点：业务类只需要关注业务逻辑本身，保证了业务类的重用性。这是代理的共有优点。 <br>
缺点：<br>
1）代理对象的一个接口只服务于一种类型的对象，如果要代理的方法很多，势必要为每一种方法都进行代理，静态代理在程序规模稍大时就无法胜任了。<br>
2）如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。

---

## 动态代理

动态代理类的源码是在程序运行期间由JVM根据反射等机制动态的生成，所以不存在代理类的字节码文件。代理类和委托类的关系是在程序运行时确定。

#### 动态代理紧密关联的Java API

###### 1. java.lang.reflect.Proxy

这是 Java 动态代理机制生成的所有动态代理类的父类，它提供了一组静态方法来为一组接口动态地生成代理类及其对象。

Proxy类的静态方法：

    // 方法 1: 该方法用于获取指定代理对象所关联的调用处理器  
    static InvocationHandler getInvocationHandler(Object proxy)   

    // 方法 2：该方法用于获取关联于指定类装载器和一组接口的动态代理类的类对象  
    static Class getProxyClass(ClassLoader loader, Class[] interfaces)   

    // 方法 3：该方法用于判断指定类对象是否是一个动态代理类  
    static boolean isProxyClass(Class cl)   

    // 方法 4：该方法用于为指定类装载器、一组接口及调用处理器生成动态代理类实例  
    static Object newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h)

###### 2. java.lang.reflect.InvocationHandler

这是调用处理器接口，它自定义了一个 invoke 方法，用于集中处理在动态代理类对象上的方法调用，通常在该方法中实现对委托类的代理访问。每次生成动态代理类对象时都要指定一个对应的调用处理器对象。

InvocationHandler的核心方法：

    // 该方法负责集中处理动态代理类上的所有方法调用。第一个参数既是代理类实例，第二个参数是被调用的方法对象  
    // 第三个方法是调用参数。调用处理器根据这三个参数进行预处理或分派到委托类实例上反射执行  
    Object invoke(Object proxy, Method method, Object[] args)   

###### 3. java.lang.ClassLoader

这是类装载器类，负责将类的字节码装载到 Java 虚拟机（JVM）中并为其定义类对象，然后该类才能被使用。Proxy 静态方法生成动态代理类同样需要通过类装载器来进行装载才能使用，
它与普通类的唯一区别就是其字节码是由 JVM 在运行时动态生成的而非预存在于任何一个 .class 文件中。每次生成动态代理类对象时都需要指定一个类装载器对象

#### 动态代理实现步骤

具体步骤是：

*   实现InvocationHandler接口创建自己的调用处理器
*   给Proxy类提供ClassLoader和代理接口类型数组创建动态代理类
*   以调用处理器类型为参数，利用反射机制得到动态代理类的构造函数
*   以调用处理器对象为参数，利用动态代理类的构造函数创建动态代理类对象

分步骤实现动态代理：

    // InvocationHandlerImpl 实现了 InvocationHandler 接口，并能实现方法调用从代理类到委托类的分派转发  
    // 其内部通常包含指向委托类实例的引用，用于真正执行分派转发过来的方法调用  
    InvocationHandler handler = new InvocationHandlerImpl(..);   

    // 通过 Proxy 为包括 Interface 接口在内的一组接口动态创建代理类的类对象  
    Class clazz = Proxy.getProxyClass(classLoader, new Class[] { Interface.class, ... });   

    // 通过反射从生成的类对象获得构造函数对象  
    Constructor constructor = clazz.getConstructor(new Class[] { InvocationHandler.class });   

    // 通过构造函数对象创建动态代理类实例  
    Interface Proxy = (Interface)constructor.newInstance(new Object[] { handler });   

Proxy类的静态方法newProxyInstance对上面具体步骤的后三步做了封装，简化了动态代理对象的获取过程。

简化后的动态代理实现：

    // InvocationHandlerImpl 实现了 InvocationHandler 接口，并能实现方法调用从代理类到委托类的分派转发  
    InvocationHandler handler = new InvocationHandlerImpl(..);   

    // 通过 Proxy 直接创建动态代理类实例  
    Interface proxy = (Interface)Proxy.newProxyInstance( classLoader,   
         new Class[] { Interface.class },  handler );   

#### 动态代理实现示例

创建自己的调用处理器：

    /**
     * 动态代理类对应的调用处理程序类
     */  
    public class SubjectInvocationHandler implements InvocationHandler {  

       //代理类持有一个委托类的对象引用  
       private Object delegate;  

       public SubjectInvocationHandler(Object delegate) {  
          this.delegate = delegate;  
       }  

       @Override  
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
          long stime = System.currentTimeMillis();   
          //利用反射机制将请求分派给委托类处理。Method的invoke返回Object对象作为方法执行结果。  
          //因为示例程序没有返回值，所以这里忽略了返回值处理  
          method.invoke(delegate, args);  
          long ftime = System.currentTimeMillis();   
          System.out.println("执行任务耗时"+(ftime - stime)+"毫秒");  

          return null;  
       }  
    }   

生成动态代理对象的工厂，工厂方法列出了如何生成动态代理类对象的步骤：

    /**
     * 生成动态代理对象的工厂.
     */  
    public class DynProxyFactory {  
       //客户类调用此工厂方法获得代理对象。  
       //对客户类来说，其并不知道返回的是代理类对象还是委托类对象。  
       public static Subject getInstance(){   
          Subject delegate = new RealSubject();  
          InvocationHandler handler = new SubjectInvocationHandler(delegate);  
          Subject proxy = null;  
          proxy = (Subject)Proxy.newProxyInstance(delegate.getClass().getClassLoader(),   
            delegate.getClass().getInterfaces(), handler);  
          return proxy;  
       }  
    }

动态代理客户类：

    public class Client {  

     public static void main(String[] args) {  

        Subject proxy = DynProxyFactory.getInstance();  
        proxy.dealTask("DBQueryTask");  

     }   

    }

---

## 应用场景

---

## 引用链接

[http://www.jianshu.com/p/6f6bb2f0ece9](http://www.jianshu.com/p/6f6bb2f0ece9)<br>
[http://layznet.iteye.com/blog/1182924](http://layznet.iteye.com/blog/1182924)<br>
