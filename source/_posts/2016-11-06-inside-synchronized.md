title: 重新认识synchronized（下）
date: 2016-11-06 12:57:14
categories: Java并发演进之路
tags: [线程同步,监视器,锁]
---
<img src="/img/synchronized.png" width="400" class="img-topic" />
synchronized既保证原子性，又保证内存可见性，是一种线程同步的方式。synchronized的实现基于JVM底层。
<!--more-->

## 线程安全与同步
线程安全表明在多线程环境中，不会有多个线程同时访问共享数据。
线程同步是线程访问类和实例字段变量，和其他共享资源的一种串行化行为，确保在同一时间只能有一个线程访问资源。举个栗子，春运火车票只剩下最后一张火车票，A，B都要抢这张火车票，怎么解决这个问题防止超卖呢？把资源保护起来，让A，B排队来买火车票。
线程安全是属性，线程同步是方式。

## 指令
synchronized同步代码块是通过monitorenter和monitorexit指令实现的，而synchronized同步方法是基于ACC_SYCHRONIZED标志，同步方法被调用时JVM会检查这个标志。monitorenter标记临界区的开始，线程执行到 monitorenter 指令时，将会尝试获取对象所对应的 monitor 的所有权；monitorexit标记临界区的结束，线程执行到 monitorexit 指令时，将释放对象所对应的 monitor 的所有权。

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