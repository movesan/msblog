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

**线程通信**：线程通信的目标是使线程间能够互相发送信号。另一方面，线程通信使线程能够等待其他线程的信号。

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
* 其他阻塞：通过调用线程的 sleep() 或 join() 发出了 I/O 请求时，线程就会进入到阻塞状态。当sleep() 状态超时，join() 等待线程终止或超时，或者 I/O 处理完毕，线程重新转入就绪状态。

**终止（dead）**: 一个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态。

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

ExecutorService、Callable、Future这个对象实际上都是属于Executor框架中的功能类。想要详细了解Executor框架的可以访问[http://www.javaeye.com/topic/366591](http://www.javaeye.com/topic/366591) ，
这里面对该框架做了很详细的解释。返回结果的线程是在JDK1.5中引入的新特征，确实很实用，有了这种特征我就不需要再为了得到返回值而大费周折了，而且即便实现了也可能漏洞百出。

可返回值的任务必须实现Callable接口，类似的，无返回值的任务必须Runnable接口。执行Callable任务后，可以获取一个Future的对象，在该对象上调用get就可以获取到Callable任务返回的Object了，
再结合线程池接口ExecutorService就可以实现传说中有返回结果的多线程了。下面提供了一个完整的有返回结果的多线程测试例子，在JDK1.5下验证过没问题可以直接使用。代码如下：

    /**
    * 有返回值的线程
    */  
    @SuppressWarnings("unchecked")  
    public class Test {  
        public static void main(String[] args) throws ExecutionException,  
            InterruptedException {  
           System.out.println("----程序开始运行----");  
           Date date1 = new Date();  

           int taskSize = 5;  
           // 创建一个线程池  
           ExecutorService pool = Executors.newFixedThreadPool(taskSize);  
           // 创建多个有返回值的任务  
           List<Future> list = new ArrayList<Future>();  
           for (int i = 0; i < taskSize; i++) {  
              Callable c = new MyCallable(i + " ");  
              // 执行任务并获取Future对象  
              Future f = pool.submit(c);  
              // System.out.println(">>>" + f.get().toString());  
              list.add(f);  
           }  
           // 关闭线程池  
           pool.shutdown();  

           // 获取所有并发任务的运行结果  
           for (Future f : list) {  
              // 从Future对象上获取任务的返回值，并输出到控制台  
              System.out.println(">>>" + f.get().toString());  
           }  

           Date date2 = new Date();  
           System.out.println("----程序结束运行----，程序运行时间【"  
             + (date2.getTime() - date1.getTime()) + "毫秒】");  
        }  
    }  


    class MyCallable implements Callable<Object> {  
        private String taskNum;  

        MyCallable(String taskNum) {  
           this.taskNum = taskNum;  
        }  

        public Object call() throws Exception {  
           System.out.println(">>>" + taskNum + "任务启动");  
           Date dateTmp1 = new Date();  
           Thread.sleep(1000);  
           Date dateTmp2 = new Date();  
           long time = dateTmp2.getTime() - dateTmp1.getTime();  
           System.out.println(">>>" + taskNum + "任务终止");  
           return taskNum + "任务返回运行结果,当前任务时间【" + time + "毫秒】";  
        }
    }

代码说明：

上述代码中Executors类，提供了一系列工厂方法用于创先线程池，返回的线程池都实现了ExecutorService接口。

创建固定数目线程的线程池:

    public static ExecutorService newFixedThreadPool(int nThreads);

创建一个可缓存的线程池，调用execute 将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程：

    public static ExecutorService newCachedThreadPool();

创建一个单线程化的Executor：

    public static ExecutorService newSingleThreadExecutor();

创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类：

    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize);

ExecutoreService提供了submit()方法，传递一个Callable，或Runnable，返回Future。如果Executor后台线程池还没有完成Callable的计算，这调用返回Future对象的get()方法，会阻塞直到计算完成。

    f.isDone(); //return true,false 无阻塞
    f.get(); // return 返回值，阻塞直到该线程运行结束

---

## Thread 方法

#### Thread对象

    1)public void start()
      使该线程开始执行；Java 虚拟机调用该线程的 run 方法
    2)public void run()
      如果该线程是使用独立的 Runnable 运行对象构造的，则调用该 Runnable 对象的 run 方法；否则，
      该方法不执行任何操作并返回
    3)public final void setName(String name)
      改变线程名称，使之与参数 name 相同
    4)public final void setPriority(int priority)
      更改线程的优先级
    5)public final void setDaemon(boolean on)
      将该线程标记为守护线程或用户线程
    6)public final void join(long millisec)
      等待该线程终止的时间最长为 millis 毫秒
    7)public void interrupt()
      中断线程
    8)public final boolean isAlive()
      测试线程是否处于活动状态

#### Thread类的静态方法

    1)public static void yield()
      暂停当前正在执行的线程对象，并执行其他线程
    2)public static void sleep(long millisec)
      在指定的毫秒数内让当前正在执行的线程休眠（暂停执行），此操作受到系统计时器和调度程序精度和准确性的影响
    3)public static boolean holdsLock(Object x)
      当且仅当当前线程在指定的对象上保持监视器锁时，才返回 true
    4)public static Thread currentThread()
      返回对当前正在执行的线程对象的引用
    5)public static void dumpStack()
      将当前线程的堆栈跟踪打印至标准错误流

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

## 守护线程

在Java线程中有两种线程，一种是User Thread（用户线程），另一种是Daemon Thread(守护线程)。
Daemon的作用是为其他线程的运行提供服务，比如说GC线程。其实User Thread线程和Daemon Thread守护线程本质上来说去没啥区别的，唯一的区别之处就在虚拟机的离开：如果User Thread全部撤离，
那么Daemon Thread也就没啥线程好服务的了，所以虚拟机也就退出了。

守护线程并非虚拟机内部可以提供，用户也可以自行的设定守护线程，方法：public final void setDaemon(boolean on) ；但是有几点需要注意：

* thread.setDaemon(true)必须在thread.start()之前设置，否则会跑出一个IllegalThreadStateException异常。你不能把正在运行的常规线程设置为守护线程。 （备注：这点与守护进程有着明显的区别，
守护进程是创建后，让进程摆脱原会话的控制+让进程摆脱原进程组的控制+让进程摆脱原控制终端的控制；所以说寄托于虚拟机的语言机制跟系统级语言有着本质上面的区别）

* 在Daemon线程中产生的新线程也是Daemon的。 （这一点又是有着本质的区别了：守护进程fork()出来的子进程不再是守护进程，尽管它把父进程的进程相关信息复制过去了，但是子进程的进程的父进程不是init进程，
所谓的守护进程本质上说就是“父进程挂掉，init收养，然后文件0,1,2都是/dev/null，当前目录到/”）

* 不是所有的应用都可以分配给Daemon线程来进行服务，比如读写操作或者计算逻辑。因为在Daemon Thread还没来的及进行操作时，虚拟机可能已经退出了。

---

## 同步与死锁

#### 线程同步

The code segments within a program that access the same object from separate, concurrent threads are called “critical sections”。这是临界区的概念。

同步的两种方式：同步块和同步方法。

每一个对象都有一个监视器，或者叫做锁。
当线程执行到synchronized的时候，检查传入的实参对象，并得到该对象的锁旗标。如果得不到，那么此线程就会被加入到一个与该对象的锁旗标相关联的等待线程池中，一直等到对象的锁旗标被归还，
池中的等待线程就会得到该锁旗标，然后继续执行下去。当线程执行完成同步代码块，就会自动释放它占有的同步对象的锁旗标。一个用于synchronized语句中的对象称为监视器，当一个线程获得了
synchronized(object)语句中的代码块的执行权，即意味着它锁定了监视器。

同步方法利用的是this所代表的对象的锁。

线程同步要时刻考虑CPU会随时切换线程的情况。同步是以牺牲程序的性能为代价的。


下面实例介绍代码块与方法间的同步。观察this的作用：

    public class ThreadDemo6 {
        public static void main(String[] args) {
           ThreadTest t = new ThreadTest();
           new Thread(t).start();
           try {
               Thread.sleep(1);//(1)
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           t.str=new String("method");
           new Thread(t).start();
        }
    }

    class ThreadTest implements Runnable {
        private int tickets = 100;
        private int flag=0;
        String str = new String("");

        public void run() {
             if (str.equals("method")) {
               while (flag==0) {
                  sale();
               }
             } else {
               while (true) {
                    synchronized (this) {//synchronized (str)//(2)
                      if (tickets > 0) {
                         try {
                             Thread.sleep(10);
                         } catch (Exception e) {
                             e.printStackTrace();
                         }
                         System.out.println(Thread.currentThread().getName()
                                + " is saling ticket " + tickets--);
                      } else return;
                    }
               }
             }
        }

        public synchronized void sale() {
             if (tickets > 0) {
               try {
                  Thread.sleep(10);
               } catch (Exception e) {
                  e.printStackTrace();
               }
               System.out.println("track in method sale.");
               System.out.println(Thread.currentThread().getName()
                      + " is saling ticket " + tickets--);
             } else flag = 1;
        }
    }

注1，如果不使主线程sleep，很可能两个新建线程都执行同步方法（sale）中的代码。因为，产生并启动第一个线程，这个线程不见得马上开始运行，CPU可能还在原来的main线程上运行，并将str变量设置为”method”，等到第一个线程真正开始运行时，此刻检查到str的值为”method”，所以它将运行sale方法。

注2，如果使用synchronized (str)，则两个线程不会同步。

**同步代码块**

    class ThreadTest implements Runnable {
        private int tickets = 100;
        String str = new String("");

        public void run() {
             while (true) {
               synchronized (str) {
                  if (tickets > 0) {
                      try {
                         Thread.sleep(10);
                      } catch (Exception e) {
                         e.printStackTrace();
                      }
                      System.out.println(Thread.currentThread().getName()
                             + " is saling ticket " + tickets--);
                  } else return;
               }
             }
        }
    }

    public class ThreadDemo2 {
        public static void main(String[] args){
           ThreadTest t=new ThreadTest();
           new Thread(t).start();
           new Thread(t).start();
           new Thread(t).start();
           new Thread(t).start();
        }
    }

**同步方法**

    class ThreadTest implements Runnable {
        private int tickets = 100;
        private int flag=0;

        public void run() {
           while (flag==0)  sale();
        }

        public synchronized void sale() {
           if (tickets > 0) {
               try {
                  Thread.sleep(10);
               } catch (Exception e) {
                  e.printStackTrace();
               }
               System.out.println(Thread.currentThread().getName()
                      + " is saling ticket " + tickets--);
           }else flag=1;
        }
    }
    public class ThreadDemo3 {
        public static void main(String[] args){
           ThreadTest t=new ThreadTest();
           new Thread(t).start();
           new Thread(t).start();
           new Thread(t).start();
           new Thread(t).start();
        }
    }

在同一个类中，使用synchronized关键字定义的若干方法，可以在多个线程之间同步，当有一个线程进入了synchronized修饰的方法（获得监视器），其他线程就不能进入同一个对象的所有使用了
synchronized修饰的方法，直到第一个线程执行完它所进入的synchronized修饰的方法为止（离开监视器）。

#### 线程死锁

线程1锁住了对象A的监视器，等待对象B的监视器，线程2锁住了对象B的监视器，等待对象A的监视器，就造成了死锁。

    class A{
        synchronized void foo(B b){
           String name=Thread.currentThread().getName();
           System.out.println(name+" enter A.foo");
           try {
               Thread.sleep(1000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           System.out.println(name+" trying to call B.last");
           b.last();
        }

        synchronized void last(){
           System.out.println(Thread.currentThread().getName()
                  +"inside A.last");
        }
    }

    class B{
        synchronized void bar(A a){
           String name=Thread.currentThread().getName();
           System.out.println(name+" enter B.bar");
           try {
               Thread.sleep(1000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           System.out.println(name+" trying to call A.last");
           a.last();
        }

        synchronized void last(){
           System.out.println(Thread.currentThread().getName()
                  +" inside B.last");
        }
    }

    public class Deadlock implements Runnable{
        A a=new A();
        B b=new B();
        Deadlock(){
             Thread.currentThread().setName("MainThread");
             new Thread(this).start();
             System.out.println("track after start");
             a.foo(b);
             System.out.println("back in main thread");
        }
        public void run(){
             System.out.println("track in run");
             Thread.currentThread().setName("RacingThread");
             b.bar(a);
             System.out.println("back in other thread");
        }
        public static void main(String[]args){
             new Deadlock();
        }
    }

结果：

    track after start
    MainThread enter A.foo
    track in run
    RacingThread enter B.bar
    MainThread trying to call B.last
    RacingThread trying to call A.last

#### 避免死锁

在有些情况下死锁是可以避免的。接下来将展示三种用于避免死锁的技术：

* 加锁顺序
* 加锁时限
* 死锁检测

**加锁顺序**

当多个线程需要相同的一些锁，但是按照不同的顺序加锁，死锁就很容易发生。

如果能确保所有的线程都是按照相同的顺序获得锁，那么死锁就不会发生。看下面这个例子：

    Thread 1:
      lock A
      lock B

    Thread 2:
       wait for A
       lock C (when A locked)

    Thread 3:
       wait for A
       wait for B
       wait for C

如果一个线程（比如线程 3）需要一些锁，那么它必须按照确定的顺序获取锁。它只有获得了从顺序上排在前面的锁之后，才能获取后面的锁。

例如，线程 2 和线程 3 只有在获取了锁 A 之后才能尝试获取锁 C(译者注：获取锁 A 是获取锁 C 的必要条件)。因为线程 1 已经拥有了锁 A，所以线程 2 和 3 需要一直等到锁 A 被释放。
然后在它们尝试对 B 或 C 加锁之前，必须成功地对 A 加了锁。

按照顺序加锁是一种有效的死锁预防机制。但是，这种方式需要你事先知道所有可能会用到的锁(译者注：并对这些锁做适当的排序)，但总有些时候是无法预知的。

**加锁时限**

另外一个可以避免死锁的方法是在尝试获取锁的时候加一个超时时间，这也就意味着在尝试获取锁的过程中若超过了这个时限该线程则放弃对该锁请求。若一个线程没有在给定的时限内成功获得所有需要的锁，则会进行回退并释放所有已经获得的锁，然后等待一段随机的时间再重试。这段随机的等待时间让其它线程有机会尝试获取相同的这些锁，并且让该应用在没有获得锁的时候可以继续运行(译者注：加锁超时后可以先继续运行干点其它事情，再回头来重复之前加锁的逻辑)。

以下是一个例子，展示了两个线程以不同的顺序尝试获取相同的两个锁，在发生超时后回退并重试的场景：

    Thread 1 locks A
    Thread 2 locks B

    Thread 1 attempts to lock B but is blocked
    Thread 2 attempts to lock A but is blocked

    Thread 1's lock attempt on B times out
    Thread 1 backs up and releases A as well
    Thread 1 waits randomly (e.g. 257 millis) before retrying.

    Thread 2's lock attempt on A times out
    Thread 2 backs up and releases B as well
    Thread 2 waits randomly (e.g. 43 millis) before retrying.

在上面的例子中，线程 2 比线程 1 早 200 毫秒进行重试加锁，因此它可以先成功地获取到两个锁。这时，线程 1 尝试获取锁 A 并且处于等待状态。当线程 2 结束时，线程 1 也可以顺利的获得这两个锁
（除非线程 2 或者其它线程在线程 1 成功获得两个锁之前又获得其中的一些锁）。

需要注意的是，由于存在锁的超时，所以我们不能认为这种场景就一定是出现了死锁。也可能是因为获得了锁的线程（导致其它线程超时）需要很长的时间去完成它的任务。

此外，如果有非常多的线程同一时间去竞争同一批资源，就算有超时和回退机制，还是可能会导致这些线程重复地尝试但却始终得不到锁。如果只有两个线程，并且重试的超时时间设定为 0 到 500 毫秒之间，
这种现象可能不会发生，但是如果是 10 个或 20 个线程情况就不同了。因为这些线程等待相等的重试时间的概率就高的多（或者非常接近以至于会出现问题）。

(译者注：超时和重试机制是为了避免在同一时间出现的竞争，但是当线程很多时，其中两个或多个线程的超时时间一样或者接近的可能性就会很大，因此就算出现竞争而导致超时后，由于超时时间一样，
它们又会同时开始重试，导致新一轮的竞争，带来了新的问题。)

这种机制存在一个问题，在 Java 中不能对 synchronized 同步块设置超时时间。你需要创建一个自定义锁，或使用 Java5 中 java.util.concurrent 包下的工具。写一个自定义锁类不复杂，但超出了本文的内容。
后续的 Java 并发系列会涵盖自定义锁的内容。

**死锁检测**

死锁检测是一个更好的死锁预防机制，它主要是针对那些不可能实现按序加锁并且锁超时也不可行的场景。

每当一个线程获得了锁，会在线程和锁相关的数据结构中（map、graph 等等）将其记下。除此之外，每当有线程请求锁，也需要记录在这个数据结构中。

当一个线程请求锁失败时，这个线程可以遍历锁的关系图看看是否有死锁发生。例如，线程 A 请求锁 7，但是锁 7 这个时候被线程 B 持有，这时线程 A 就可以检查一下线程 B 是否已经请求了线程 A 当前所持有的锁。如果线程 B 确实有这样的请求，那么就是发生了死锁（线程 A 拥有锁 1，请求锁 7；线程 B 拥有锁 7，请求锁 1）。

当然，死锁一般要比两个线程互相持有对方的锁这种情况要复杂的多。线程 A 等待线程 B，线程 B 等待线程 C，线程 C 等待线程 D，线程 D 又在等待线程 A。线程 A 为了检测死锁，它需要递进地检测所有被 B 请求的锁。从线程 B 所请求的锁开始，线程 A 找到了线程 C，然后又找到了线程 D，发现线程 D 请求的锁被线程 A 自己持有着。这是它就知道发生了死锁。

下面是一幅关于四个线程（A,B,C 和 D）之间锁占有和请求的关系图。像这样的数据结构就可以被用来检测死锁。

![img](/img/post-in/post-thread-3.png)

那么当检测出死锁时，这些线程该做些什么呢？

一个可行的做法是释放所有锁，回退，并且等待一段随机的时间后重试。这个和简单的加锁超时类似，不一样的是只有死锁已经发生了才回退，而不会是因为加锁的请求超时了。虽然有回退和等待，但是如果有大量的线程竞争同一批锁，它们还是会重复地死锁（编者注：原因同超时类似，不能从根本上减轻竞争）。

一个更好的方案是给这些线程设置优先级，让一个（或几个）线程回退，剩下的线程就像没发生死锁一样继续保持着它们需要的锁。如果赋予这些线程的优先级是固定不变的，同一批线程总是会拥有更高的优先级。为避免这个问题，可以在死锁发生的时候设置随机的优先级。

## 多线程的使用

有效利用多线程的关键是理解程序是并发执行而不是串行执行的。例如：程序中有两个子系统需要并发执行，这时候就需要利用多线程编程。
通过对多线程的使用，可以编写出非常高效的程序。不过请注意，如果你创建太多的线程，程序执行的效率实际上是降低了，而不是提升了。

请记住，上下文的切换开销也很重要，如果你创建了太多的线程，CPU 花费在上下文的切换的时间将多于执行程序的时间！

## 引用链接

[Java中的多线程你只要看这一篇就够了 - 知米丶无忌 简书](http://www.jianshu.com/p/40d4c7aebd66)<br>
[Java多线程干货系列—（一）Java多线程基础 - 嘟嘟MD 简书](http://www.jianshu.com/p/4e2343cf747b)<br>
[Java 多线程编程 - 菜鸟教程](http://www.runoob.com/java/java-multithreading.html)<br>
[Java并发性和多线程 - 极客学院](http://wiki.jikexueyuan.com/project/java-concurrent/thread-communication.html)<br>
[Java多线程之同步与死锁 - 谢芳](http://www.cnblogs.com/xiefang1980/archive/2008/04/21/1163882.html)<br>
