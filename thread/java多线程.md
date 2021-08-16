# java多线程详解

- 线程简介
- 线程实现（重点）
- 线程状态
- 线程同步（重点）
- 线程通信问题
- 高级主题



### 线程、进程、多线程



### Prcessor和Thread

- 说起进程，就不得不说一下程序。程序是指令和数据的有序集合，其本身没有仍和运行的含义，是一个静态的概念
- 而进程是执行程序的一次执行过程，他是一个动态的概念。是系统资源分配的基本单位
- 通常一个进程可以包含多个线程，当然一个程序至少有一个线程，不然没有办法运行程序没有存在的意义。线程是CPU调度和执行的基本单位

注意：很多多线程是模拟出来的，真正的多线程指的是多个CPU多个核心。

本章的核心概念

- 线程是独立的执行路径
- 在程序运行之时，即使自己没有创建看线程，后台也会有多个线程，如main线程和gc线程
- main() 称之为主线程，为系统的入口，用于执行整个程序
- 线程的运行由调度器进行安排，与操作系统密切相关，不能认为干预
- 对同一份资源操作时，会存在资源抢夺问题，需要加入并发控制。
- 线程会带来额外的开销
- 每个线程都在自己的工作内存交互，内存控制不当会造成数据不一致





## 线程创建

​	Thread、Runnable、Callable

三种创建方式

![image-20210811093105229](C:\Users\Coder guo\AppData\Roaming\Typora\typora-user-images\image-20210811093105229.png)





Thread开启方法：

1. 继承Thread类

- 自定义线程类集成Thread类
- 重写run方法
- 创建线程对象，调用start() 方法启动线程



2.实现runnable接口（推荐使用Runnable，因为java是单继承的）

- 定义类实现Runnable方法
- 实现run()方法，编写线程执行体
- 创建线程对象，调用start() 方法启动线程



3.实现callable接口

```java
/**
 * 使用 callable实现接口
 * 1.创建执行任务
 * 2.执行提交
 * 3.获取结果
 * 4.关闭服务
 */
  //* 1.创建执行任务
        ExecutorService ser = Executors.newFixedThreadPool(3);

        //* 2.执行提交
        Future<Boolean> submit1 = (Future<Boolean>) ser.submit(t1);
        Future<Boolean> submit2 = (Future<Boolean>) ser.submit(t2);
        Future<Boolean> submit3 = (Future<Boolean>) ser.submit(t3);

        //* 3.获取结果
        Boolean r1 = submit1.get();
        Boolean r2 = submit2.get();
        Boolean r3 = submit3.get();
        System.out.println(r1 + " " + r2 + " " + r3);

        //* 4.关闭服务
        ser.shutdown();
```



总结：

线程开启不一定立即执行，由cpu调度

继承Thread类

- 子类继承Thread类具备多线程能力
- 启动线程：子类对象，start（）
- 不建议使用：子类对象，start()

实现runnable接口：

- 实现runnable接口的类具备有多线程的能力
- 启动线程：传入目标对象+Thread对象.start()
- 推荐使用：避免单继承局限性，灵活方便，方便同一个对象被多个线程使用

实现callable的优点

- 类实现了callable后，实现其call方法
- 优点是：有返回值、能够抛出异常
- 但是实现更加复杂，需要完成以下四步
   * 1.创建执行任务
   * 2.执行提交
   * 3.获取结果
   * 4.关闭服务



## 线程中的并发问题

```java
package com.guo.thread;

// 并发问题： 当多个线程操作同一个资源的情况下，线程不安全，数据出现紊乱
public class TestThread5 implements Runnable{
    private int ticketsNum = 10;
    @Override
    public void run() {
        while (true) {
            if(ticketsNum <= 0) break;
            System.out.println(Thread.currentThread().getName() + "--> 拿到第" + ticketsNum-- + "张票");
        }
    }

    public static void main(String[] args) {
        TestThread5 t1 = new TestThread5();
        TestThread5 t2 = new TestThread5();
        TestThread5 t3 = new TestThread5();

        new Thread(t1, "小明").start();
        new Thread(t2, "老师").start();
        new Thread(t3, "黄牛").start();
    }
}
```



## 静态代理 

```java
package com.guo.proxystatic;

public class ProxyStatic {
    public static void main(String[] args) {
        WeddingCompany weddingCompany = new WeddingCompany(new You());
        weddingCompany.HappyMarry();

    }
}

interface Marry {
    void HappyMarry();
}

class You implements Marry {
    @Override
    public void HappyMarry() {
        System.out.println("Coder 要结婚了，超级开心");
    }
}

class WeddingCompany implements Marry {
    private You you;

    public WeddingCompany(You you) {
        this.you = you;
    }

    private void before() {
        System.out.println("before wedding");
    }

    private void after() {
        System.out.println("after wedding");
    }
    @Override
    public void HappyMarry() {
        before();
        you.HappyMarry();
        after();

    }
}
```





## lamda表达式

- 避免匿名内部类定义过多

- 其实质属于函数式编程的概念

  ```java
  (param) -> expresssion
  (param) -> statement
  (param) -> {statement}
  ```

  

  实例：

  ```java
  	new Thread(()->System.out.println("before wedding")).start();
  ```

  

什么是函数式接口？

​	函数接口的定义: 任何接口，如果只包含唯一一个抽象方法，那么他就是一个函数式接口

![image-20210813150943219](C:\Users\Coder guo\AppData\Roaming\Typora\typora-user-images\image-20210813150943219.png)



总结lamda表达式：

- lamda表达式的只能有一行衣阿华那个代码的情况下，简化为一行，否则使用代码块包裹
- 前提是函数式接口（接口里只有一个函数）
- 多个参数也能去掉参数类型，但是如果要去掉类型的话需要都去除



##  线程停止

线程的五大状态

**![image-20210813155805977](C:\Users\Coder guo\AppData\Roaming\Typora\typora-user-images\image-20210813155805977.png)**



- 建议线程正常停止--利用次数，不建议死循环
- 建议使用标志位--设置一个标志位来终止线程
- 不要使用stop或者destory等过时的方法。

  样例：

```java
package com.guo.state;

import com.guo.thread.TestThread2;

public class TestStop implements Runnable {
    private boolean flag = true;

    @Override
    public void run() {
        while(flag) {
            System.out.println("Test thread is running ......");
        }
    }

    private void stop() {
        this.flag = false;
    }

    public static void main(String[] args) {
        TestStop testStop = new TestStop();
        new Thread(testStop).start();

        for (int i = 0; i < 5000; i++) {
            System.out.println("this is main thread--->" + i);
            if(i == 4900) {
                System.out.println("线程该停止啦--->main" + i);
                testStop.stop();
            }
        }
    }

}
```





## 线程休眠

- sleep事件指定当前线程的毫秒数
- sleep存在异常InterruptedException
- sleep时间达到后线程进入就绪状态
- sleep可以模拟网络延时，倒计时
- 每一个对象都有一个锁，sleep不会释放锁





## 线程礼让

- 礼让线程，让当前正在执行的线程，但是不阻塞
- 线程从运行状态转化为就绪状态
- 让cpu重新调度，礼让不一定成功 



## Join

- Join合并线程，待此线程完成后，再执行其他线程（此时其他线程阻塞）
- 可以想象成插队





### 线程状态观测

Thread.State

- NEW 尚未启动的线程处于此状态
- RUNNALE：在Java虚拟机中执行的线程处于此状态
- BLOCKED：被阻塞等待监视器锁定的线程处于此状态
- WAITING： 正在等待另一个线程执行特定动作的线程处于此状态
- TIME_WAITING：在等待另一个线程的执行特定动作的线程处于此状态
- TEMINATED： 已经退出的线程处于此状态

死亡的线程无法再次启动



### 线程的优先级

- java提供一个线程调度控制器来监控程序中启动后进入就绪状态的所有线程，线程调度器按照使用优先级决定应该调度哪个线程

- 线程优先级用数字表示

- 使用getPriority和setPriority改变和获取优先级

  优先级的设置建议在start调度之前，且并不是优先级越高就一定先执行



### 守护线程（daemon）

- 线程分为用户线程 和 守护线程
- 虚拟机必须确保用户线程执行完毕
- 举例：后台日志记录，监控内存，垃圾回收等待





## 线程同步机制

多个线程操作同一个资源造成的。

	由于同一个进程的多个线程共享一块存储区域，带来方便的同时，也带来了访问冲突。为了保证数据在方法中被访问时的正确性，在访问时加入锁机制synchronized，当一个线程获得兑现的排他锁，独占资源，其他的线程必须等待，使用后释放锁即可。解决同步存在以下问题

- 一个线程持有锁回导致所有其他有需要的锁挂起
- 在多线程竞争下，加锁，释放锁可能会导致比较多的上下文切换和调度延时，引起性能问题
- 如果一个优先级高的线程等待一个优先级低的线程释放锁，肯能导致优先级倒置，引起性能问题



#### 同步方法

		由于我们可以通过private关键字来保证数据对象只能被方法访问，所以我们需要只针对方法提出一套机制，这套机制就是synchronized关键字，它包括两个用法 synchronized方法和synchronized块
	
		synchronized方法控制对对象的访问，每个对象都有一把锁，每个synchronized方法都必须获得调用该方法的锁才能执行相关的操作，否则线程会进行阻塞。方法一旦执行就独占该锁。直到该方法返回猜释放锁，后面阻塞的线程才能获得这个锁，继续执行。
	
		缺点：可能会影响方法执行的效率。

同步块

- synchronized（obj） {}
- Obj称之为同步监视器
  - Obj可以时任何对象，但是推荐使用共享资源作为同步监视器
  - 同步方法中无需指定同步监视器，因为同步方法就是this，就是这个对象本身，或者是class
- 同步监视器执行过程
  1. 第一个线程访问，锁定同步监视器，执行其中代码
  2. 第二个线程访问，发现同步监视器访问，无法访问
  3. 第一个线程访问完毕，解锁同步监视器
  4. 第二个线程访问，发现同步监视器没有锁，然后锁定并访问



### 死锁

```mag
	多个线程各自占有一些共享资源，并且相互等待其他线程占有的资源才能运行，而导致两个或者两个以上线程都在等待对方释放资源，都停止执行的情形。某一个同步块同时拥有两个以上对象的锁时，可能会发生死锁的现象。
```



#### 死锁避免方法

- 互斥条件：一个资源一次只能被一个进程使用
- 请求和保持条件： 一个进程因请求资源而阻塞，对已经获得的资源保持不放
- 不剥夺条件： 进程已经获得资源但是未使用完成之前，不能强行剥夺
- 循环等待条件：若干进程之间形成的一种头尾相接的资源循环等待关系。



## Lock锁

- 从JDK5开始，java提供了更强大的线程同步机制-通过显示定义同步锁对象来实现同步。同步锁使用Lockk对象充当
- java.util.concurrent.Lock接口时控制多个线程共享资源进行访问的工具。锁提供了对共享资源的独占访问，每次只能有一个线程对Lock对象枷加锁，线程开始访问共享资源之前应先获得锁对象。
- ReentrantLock类实现了Lock，它拥有与synchronized相同的并发性和内存语义，在实现线程安全的控制中，比较常用的ReentrantLock，可以显示加锁锁，释放锁



### Lock和synchronized对比

- Lock是显示锁（需要手动开启和关闭）synchronized是隐式锁，除了作用域自动释放。
- Lock只有代码块锁，synchronized有代码块锁和方法锁
- 使用lock锁，JVM将花费较少的时间来调度线程，性能更好。并且具有更好的拓展性。（提供更多子类）
- 优先的使用顺序： Lock -》 同步代码块 -》 同步方法



## 线程协作

​	生产者消费者模式，两种解决的方法

- 管程法
- 信号灯法



## 使用线程池

- 背景经常创建和销毁，使用量特别大的资源，比如并发情况下的线程。对性能影响很大。
- 思路：提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中。可以避免频繁的创建销毁，实现重复利用。
- 好处：
  - 提高响应速度
  - 降低资源消耗
  - 便于线程管理





## 总结

​	学习java多线程课程，了解线程和进程之间的关系。

​	第一个阶段-----学习到了实现线程的三种方法，第一种是实现runnable接口重写run方法，第二种是继承thread接口重写run方法，第三种是实现callable接口。

​	线程中并发问题的由来，多个线程同一个时间节点访问一个共享资源导致出现脏数据情况。解决办法就是两种，同步和加锁。

​	线程的启动是调用线程的start方法，线程的启动使用代理模式的静态代理。

​	静态代理简述：由两个类通过实现同一个接口，将一个类的实例对象作为对象传入到另一个对象中，接受传入数据的对象可以在其实现的方法进行进一步的封装。

​	了解了java中Lamda表达式的基本写法。和前端ES6语法的lamda写法表达式类似。

​	

​	第二个阶段---了解线程具有的状态，主要状态为 就绪状态，阻塞状态，休眠状态，运行状态，死亡状态。以及线程的进入这些状态一些方法。

- sleep方法让线程休眠多少毫秒，
- join方法设置线程加入，
- 线程的停止官方不建议使用stop和destory方法，而是建议使用标志位来进行控制终止。
- 线程setPriority(get)可以设置和获取当前线程优先级。

守护线程，是从一开始使用程序运行开始就一直执行的线程，jvm不能自动处理这类线程，它会一致伴随直到进程运行结束。类似于上帝一致伴随在身边。使用setDaemon方法来设置守护线程。



第三阶段，同步（synchronized）和加锁（Lock），synchronized关键字可以在方法和块上使用，而加锁只能在代码块中使用。但是相比较synchronized来说，加锁会使jvm的是运行速度更快。因此，使用的优先级为 lock 》 synchronized块 》 synchronized方法



第四阶段，消费者和生产者模型，使用线程进行模拟。了解到了两种解决办法：

- 管程法（使用缓存区域解决）
- 信号灯法（使用flag旗帜来解决）

了解到了使用线程池的基本使用，使用ServiceExcutor类即可。使用Excutors.newFixedThreadPool创建线程池。

