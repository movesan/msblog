---
layout:     post
title:      "Java核心技术---多线程（THREAD）"
subtitle:   " Java Thread "
date:       2017-02-16 22:00:00
author:     "Movesan"
header-img: "img/post-bg/post-bg-java.jpg"
catalog: true
tags:
    - Java

---

## 引言

多线程并发编程是Java编程中重要的一块内容，程序的每一部分都称作一个线程，并且每个线程定义了一个独立的执行路径。一个线程不能独立的存在，它必须是进程的一部分。一个进程一直运行，
直到所有的非守候线程都结束运行后才能结束。

多线程能满足程序员编写高效率的程序来达到充分利用 CPU 的目的。

---

## 概念

学习多线程编程之前，我们需要先了解下有关多线程的概念。

**进程**：执行中的程序一个进程至少包含一个线程

**线程**：进程中负责程序执行的执行单元。线程本身依靠程序进行运行，线程是程序中的顺序控制流，只能使用分配给程序的资源和环境

**单线程**：程序中只存在一个线程，实际上主方法就是一个主线程

**多线程**：指的是这个程序（一个进程）运行时产生了不止一个线程

**并行与并发**：<br>
并行：多个cpu实例或者多台机器同时执行一段处理逻辑，是真正的同时。<br>
并发：通过cpu调度算法，让用户看上去同时执行，实际上从cpu操作层面不是真正的同时。并发往往在场景中有公用的资源，那么针对这个公用的资源往往产生瓶颈，我们会用TPS或者QPS来反应这个系统的处理能力。<br>

下图描述了并行与并发的区别：

![img](/img/post-in/post-thread-1.jpg)

**线程安全**：经常用来描绘一段代码。指在并发的情况之下，该代码经过多线程使用，线程的调度顺序不影响任何结果。这个时候使用多线程，我们只需要关注系统的内存，cpu是不是够用即可。
反过来，线程不安全就意味着线程的调度顺序会影响最终结果，如不加事务的转账代码：

    void transferMoney(User from, User to, float amount){
      to.setMoney(to.getBalance() + amount);
      from.setMoney(from.getBalance() - amount);
    }

**同步**：Java中的同步指的是通过人为的控制和调度，保证共享资源的多线程访问成为线程安全，来保证结果的准确。如上面的代码简单加入@synchronized关键字。在保证结果准确的同时，
提高性能，才是优秀的程序。线程安全的优先级高于性能。

---

## 线程的生命周期

线程是一个动态执行的过程，它也有一个从产生到死亡的过程。下图显示了一个线程完整的生命周期：

![img](/img/post-in/post-thread-2.jpg)

线程在一个动态的生命周期中，会有不同的状态：

**创建（new）**: 使用 new 关键字和 Thread 类或其子类建立一个线程对象后，该线程对象就处于新建状态。它保持这个状态直到程序 start() 这个线程。<br>
**就绪（runnable）**: 当线程对象调用了start()方法之后，该线程就进入就绪状态。就绪状态的线程处于就绪队列中，要等待JVM里线程调度器的调度。<br>
**运行（running）**: 如果就绪状态的线程获取 CPU 资源，就可以执行 run()，此时线程便处于运行状态。处于运行状态的线程最为复杂，它可以变为阻塞状态、就绪状态和死亡状态。<br>
**阻塞（blocked）**: 如果一个线程执行了sleep（睡眠）、suspend（挂起）等方法，失去所占用资源之后，该线程就从运行状态进入阻塞状态。在睡眠时间已到或获得设备资源后可以重新进入就绪状态。可以分为三种：<br>
* 等待阻塞：运行状态中的线程执行 wait() 方法，使线程进入到等待阻塞状态。
* 同步阻塞：线程在获取 synchronized 同步锁失败(因为同步锁被其他线程占用)。
* 其他阻塞：通过调用线程的 sleep() 或 join() 发出了 I/O 请求时，线程就会进入到阻塞状态。当sleep() 状态超时，join() 等待线程终止或超时，或者 I/O 处理完毕，线程重新转入就绪状态。<br>
*终止（dead）*: 一个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态。

---

## 线程的优先级

每一个 Java 线程都有一个优先级，这样有助于操作系统确定线程的调度顺序。
Java 线程的优先级是一个整数，其取值范围是 1 （Thread.MIN_PRIORITY ） - 10 （Thread.MAX_PRIORITY ）。
默认情况下，每一个线程都会分配一个优先级 NORM_PRIORITY（5）。
具有较高优先级的线程对程序更重要，并且应该在低优先级的线程之前分配处理器资源。但是，线程优先级不能保证线程执行的顺序，而且非常依赖于平台。

---

## 上下文切换

对于单核CPU来说（对于多核CPU，此处就理解为一个核），CPU在一个时刻只能运行一个线程，当在运行一个线程的过程中转去运行另外一个线程，这个叫做线程上下文切换（对于进程也是类似）。

由于可能当前线程的任务并没有执行完毕，所以在切换时需要保存线程的运行状态，以便下次重新切换回来时能够继续切换之前的状态运行。举个简单的例子：比如一个线程A正在读取一个文件的内容，正读到文件的一半，
此时需要暂停线程A，转去执行线程B，当再次切换回来执行线程A的时候，我们不希望线程A又从文件的开头来读取。

因此需要记录线程A的运行状态，那么会记录哪些数据呢？因为下次恢复时需要知道在这之前当前线程已经执行到哪条指令了，所以需要记录程序计数器的值，另外比如说线程正在进行某个计算的时候被挂起了，
那么下次继续执行的时候需要知道之前挂起时变量的值时多少，因此需要记录CPU寄存器的状态。所以一般来说，线程上下文切换过程中会记录程序计数器、CPU寄存器状态等数据。

说简单点的：对于线程的上下文切换实际上就是 存储和恢复CPU状态的过程，它使得线程执行能够从中断点恢复执行。

虽然多线程可以使得任务执行的效率得到提升，但是由于在线程切换时同样会带来一定的开销代价，并且多个线程会导致系统资源占用的增加，所以在进行多线程编程时要注意这些因素。

---

## 线程的创建方式

Java 提供了三种创建线程的方法：

* 通过实现 Runnable 接口；
* 通过继承 Thread 类本身；
* 通过 Callable 和 Future 创建线程。

#### 通过继承 Thread 来创建线程

在java.lang包中定义, 继承Thread类必须重写run()方法

    class MyThread extends Thread{
        private static int num = 0;

        public MyThread(){
            num++;
        }

        @Override
        public void run() {
            System.out.println("主动创建的第"+num+"个线程");
        }
    }

创建好了自己的线程类之后，就可以创建线程对象了，然后通过start()方法去启动线程。注意，不是调用run()方法启动线程，run()方法中只是定义需要执行的任务，如果调用run方法，
即相当于在主线程中执行run方法，跟普通的方法调用没有任何区别，此时并不会创建一个新的线程来执行定义的任务。

    public class Test {
        public static void main(String[] args)  {
            MyThread thread = new MyThread();
            thread.start();
        }
    }
    class MyThread extends Thread{
        private static int num = 0;
        public MyThread(){
            num++;
        }
        @Override
        public void run() {
            System.out.println("主动创建的第"+num+"个线程");
        }
    }

在上面代码中，通过调用start()方法，就会创建一个新的线程了。为了分清start()方法调用和run()方法调用的区别，请看下面一个例子：

    public class Test {
        public static void main(String[] args)  {
            System.out.println("主线程ID:"+Thread.currentThread().getId());
            MyThread thread1 = new MyThread("thread1");
            thread1.start();
            MyThread thread2 = new MyThread("thread2");
            thread2.run();
        }
    }


    class MyThread extends Thread{
        private String name;

        public MyThread(String name){
            this.name = name;
        }

        @Override
        public void run() {
            System.out.println("name:"+name+" 子线程ID:"+Thread.currentThread().getId());
        }
    }

运行结果：

    主线程ID：1
    name：thread2 子线程ID：1
    name：thread1 子线程ID：8

从输出结果可以得出以下结论：

* thread1和thread2的线程ID不同，thread2和主线程ID相同，说明通过run方法调用并不会创建新的线程，而是在主线程中直接运行run方法，跟普通的方法调用没有任何区别；
* 虽然thread1的start方法调用在thread2的run方法前面调用，但是先输出的是thread2的run方法调用的相关信息，说明新线程创建的过程不会阻塞主线程的后续执行。   

#### 通过实现 Runnable 接口来创建线程

在Java中创建线程除了继承Thread类之外，还可以通过实现Runnable接口来实现类似的功能。实现Runnable接口必须重写其run方法。

下面是一个例子：

    public class Test {
        public static void main(String[] args)  {
            System.out.println("主线程ID："+Thread.currentThread().getId());
            MyRunnable runnable = new MyRunnable();
            Thread thread = new Thread(runnable);
            thread.start();
        }
    }
    class MyRunnable implements Runnable{
        public MyRunnable() {
        }

        @Override
        public void run() {
            System.out.println("子线程ID："+Thread.currentThread().getId());
        }
    }

Runnable的中文意思是“任务”，顾名思义，通过实现Runnable接口，我们定义了一个子任务，然后将子任务交由Thread去执行。注意，这种方式必须将Runnable作为Thread类的参数，然后通过Thread的start方法
来创建一个新线程来执行该子任务。如果调用Runnable的run方法的话，是不会创建新线程的，这根普通的方法调用没有任何区别。

事实上，查看Thread类的实现源代码会发现Thread类是实现了Runnable接口的。

在Java中，这2种方式都可以用来创建线程去执行子任务，具体选择哪一种方式要看自己的需求。直接继承Thread类的话，可能比实现Runnable接口看起来更加简洁，但是由于Java只允许单继承，所以如果自定义类需
要继承其他类，则只能选择实现Runnable接口。

#### 通过 Callable 和 Future 创建线程
