title: 重新认识synchronized（下）
date: 2016-11-06 12:57:14
categories: Java并发演进之路
tags: [线程同步,监视器,锁]
photos:
  - /img/synchronized-pause.png
---
synchronized既保证原子性，又保证内存可见性，是一种线程同步的方式，是锁机制的一种java实现。synchronized的实现基于JVM底层，JVM是基于monitor实现的，而monitor的实现依赖于操作系统的互斥实现。
<!--more-->

## 语义
synchronized语义是同步，但同步有两层含义：
1. 互斥，即锁的特点。同一时间只能有一个线程持有监视器，因此一旦线程进入监视器保护的代码块（即临界区），其他线程是不允许监视器保护的代码块，直到前一个线程退出代码块。互斥阻止了其他线程看到对象不一致的状态，与原子性有相同的语义。
2. 可见。synchronized保证进入同步代码块的线程，都可以看到由同一个锁保护的之前所有的修改效果。原因是：在释放监视器时（即退出同步代码块），会将工作内存中未映射到主内存的工作拷贝，强制刷新回主内容。在获取监视器是（即进入同步代码块时），监视器会使本地内存失效，强制从主内存拷贝到工作内存。

互斥保证在线程退出前，所有对象状态变更都对其他线程不可见；可见保证在线程进入同步代码块时，可以看到上一个线程对对象状态变更的最终状态。

## 线程安全与同步
线程安全表明在多线程环境中，不会有多个线程同时访问共享数据。
线程同步是线程访问类和实例字段变量，和其他共享资源的一种串行化行为，确保在同一时间只能有一个线程访问资源。举个栗子，春运火车票只剩下最后一张火车票，A，B都要抢这张火车票，怎么解决这个问题防止超卖呢？把资源保护起来，让A，B排队来买火车票。
线程安全是属性，线程同步是方式。

## 指令
synchronized同步代码块是通过monitorenter和monitorexit指令实现的，而synchronized同步方法是基于ACC\_SYCHRONIZED标志，同步方法被调用时JVM会检查这个标志。monitorenter标记临界区的开始，线程执行到 monitorenter 指令时，将会尝试获取对象所对应的 monitor 的所有权；monitorexit标记临界区的结束，线程执行到 monitorexit 指令时，将释放对象所对应的 monitor 的所有权。

```java
public class SynchronizedMethod {
    public synchronized void methodA() {
        System.out.println("MethodA start");

    }
}
```

将这段代码通过 `javap -c` 反编译一下，重点关注一下编译后的第3行和第13行。

```java
Compiled from "SynchronizedTest.java"
public class com.memory.SynchronizedTest {
  public com.memory.SynchronizedTest();
    Code:
       0: aload_0       
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return        

  public void methodA();
    Code:
       0: aload_0       
       1: dup           
       2: astore_1      
       3: monitorenter  
       4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       7: ldc           #3                  // String MethodA start
       9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      12: aload_1       
      13: monitorexit   
      14: goto          22
      17: astore_2      
      18: aload_1       
      19: monitorexit   
      20: aload_2       
      21: athrow        
      22: return        
    Exception table:
       from    to  target type
           4    14    17   any
          17    20    17   any
}
```

## 锁的种类
![java synchronized锁升级](/img/synchronized-lock-upgrade.png "java synchronized锁升级")
JDK1.6中对synchronized优化引入了偏向锁，轻量级锁，重量级锁。锁的升级过程是单方向的，只允许从低到高升级，不允许降级。

重量级锁（Heavyweight Lock）是将程序运行交出控制权，将线程挂起，由操作系统来负责线程间的调度，负责线程的阻塞和执行。这样会出现频繁地对线程运行状态的切换，线程的挂起和唤醒，消耗大量的系统资源，导致性能低下。

轻量级锁（lightweight Locking）是相对于重量级锁而言的，在synchronized实现中使用自旋的方式，实际是通过CPU自旋等待的方式替代线程切换，竞争的线程不会因此而阻塞，避免阻塞唤醒造成的CPU负荷。采用自旋的方式有利有弊，当锁占用的时间较短时，较少次数的自旋等待就可以获取锁；但在锁占用的时间较长时，自旋会白白浪费大量的CPU资源。因此自旋的次数有一定要在限定之内，自旋失败就会立即将锁升级为重量级锁，称为锁膨胀。

偏向锁（Biased Locking ）从字面含义是这把锁是有私心的，会倾向于上次访问的线程。Hotspot的作者在他的论文《QRL-OpLocks-BiasedLocking》中阐述到，研究发现大多数情况下不存在多线程争夺共享资源，而且总是由同一线程多次获得，考虑到CAS （Compare-And-Swap）指令在获取Java监视器时会造成较大的CPU延迟，为了让线程获得锁的代价更低而引入了偏向锁。

## 锁标记的位置
64位虚拟机中，标记字段（Mark Word）中包含哈希吗（HashCode，存放31bits对象的hashcode值），GC分代年龄（Generational GC Age，4bits，因此分代年龄最高为15），偏向线程ID，偏向锁标记。
synchronized锁的四个状态：无锁状态，偏向锁，轻量级锁和重量级锁，在Mark Word中对应不同的字段。
![java synchronized不同级别锁中的Mark Word](/img/synchronized-markword.png "java synchronized不同级别锁中的Mark Word")

我是葛一凡，希望对你有用。
![微信公众号](/img/qrcode.jpg "微信公众号")

## 参考
1. [JSR 133 (Java Memory Model) FAQ](http://www.cs.umd.edu/\~pugh/java/memoryModel/jsr-133-faq.html)
2. [Java内存模型(一)](http://www.cloudchou.com/softdesign/post-631.html)
3. [Java 锁机机制——浅析 Synchronized](http://blog.csdn.net/wwh578867817/article/details/52004265)
4. [Biased Locking in HotSpot](https://blogs.oracle.com/dave/entry/biased_locking_in_hotspot)
5. [Java Synchronised机制](https://blog.dreamtobe.cn/2015/11/13/java_synchronized/)
6. [深入JVM锁机制1-synchronized](http://blog.csdn.net/chen77716/article/details/6618779)
7. [多线程之：偏向锁，轻量级锁，重量级锁](http://www.cnblogs.com/shangxiaofei/p/5569879.html)
8. [JVM内部细节之一：synchronized关键字及实现细节（轻量级锁Lightweight Locking）](http://www.cnblogs.com/javaminer/p/3889023.html)
9. [java锁优化](http://luojinping.com/2015/07/09/java%E9%94%81%E4%BC%98%E5%8C%96/)
10. [Java并发编程：Synchronized底层优化（偏向锁、轻量级锁）](http://www.cnblogs.com/paddix/p/5405678.html)
11. [虚拟机中的锁优化简介（适应性自旋/锁粗化/锁削除/轻量级锁/偏向锁）](http://icyfenix.iteye.com/blog/1018932)
12. [JVM内部细节之二：偏向锁（Biased Locking）](http://www.cnblogs.com/javaminer/p/3892288.html)
13. [小议偏向锁](http://xieyj.iteye.com/blog/322089)
14. [synchronized、锁、多线程同步的原理是咋样的](http://www.jianshu.com/p/5dbb07c8d5d5)
15. [Java中的锁](http://www.importnew.com/19472.html)
16. [markOop.hpp](http://hg.openjdk.java.net/jdk7/jdk7/hotspot/file/9b0ca45cd756/src/share/vm/oops/markOop.hpp)
17. [5 Things You Didn't Know About Synchronization in Java and Scala](http://blog.takipi.com/5-things-you-didnt-know-about-synchronization-in-java-and-scala/)
18. 论文：The Smart Energy Management of Multithreaded Java Applications on Multi-Core Processors


