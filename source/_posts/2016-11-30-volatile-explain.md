title: volatile语义
date: 2016-11-30 16:03:08
categories: Java并发演进之路
tags: [主内存,工作内存,同步,共享变量]
---
<img src="/img/volatile-visible.png" width="300" class="img-topic" />
volatile在Java内存模型（JMM）中，保证共享变量对所有线程可见，但不保证原子性。volatile语义是同步，通过共享变量的方式，完成线程间的通信。
<!--more-->

## 为什么需要volatile
Java内存模型中抽象、简化了计算机物理设备，分成工作内存和主内存，线程有各自的工作内存，却共享主内存。如果要把Java内存模型与物理设备映射起来的话，L1，L2 Cache可以视为工作内存，而L3 Cache视为主内存。线程执行指令时，会优先选择距离 CPU 较近的位置的工作内存中使用，而不会从读写速度较慢的主内存中，我称之为“就近原则”。当线程指令执行完后，赋值给工作内存，如果不回写到主内存，或者通知其他线程，其他线程是无法知晓变量已经修改，仍然会使用曾经缓存在工作内存中的变量，这就造成了缓存不一致的问题，Java使用volatile解决这种问题。**volatile保证指令赋值完后的变量立即同步回主内存中，声明并通知其他线程当前赋值的变量已经失效，其他线程在下次使用时会放弃工作内存中变量，使用主内存中的变量。**这样就完成了线程间对于volatile修饰的变量的通信。

## 可见性
执行引擎只与工作内存交互，再有工作内存与主内存交互。站在执行引擎的角度，与工作内存操作完成即表示指令执行完，但是什么时候工作内存会将结果刷新回主内存却不可预测。Java线程间的通信是通过共享内存的方式，线程A如果想通知其他所有线程（线程B，线程C）对于变量f的变化情况，需要满足两点：
1. 将变量回写到主内存中
2. 执行引擎读取时强制从主内存中加载
在增加了增加了L1、L2 Cache之后，CPU何时将变量从独享缓存刷新会共享内存，独享缓存是否从共享内存加载变量，时间上都是不可确定的，这就造成了缓存不一致的问题。

可见性的语义是线程对变量更新操作后，其他线程是可以获知变量的变更情况。
![工作内存和主内存关系](/img/volatile-jmm.png "工作内存和主内存关系")

## 原子性
原子操作是一个或多个**不可中断**的操作，要么一次性完全执行完毕，要么就不执行，最终状态不存在有些操作执行完，有些操作没有执行，在外部看来是不可分割的整体（比如化学中的原子，当然原子也是可以再分割的，不过站在分子层面，原子是最小的不可分割的），原子操作关注的是不被线程调度器中断的操作。

原子性操作是不会出现线程交替执行的情况，如果出现线程交替，则说明操作被线程调度器中断。在Java内存模型中，原子性保证你获取的变量要么是初始值，要么是被某一个线程写入的值，而不会是有多个线程同一时间写入而产生的混乱结果，long或double类型除外（因为变量的前32位可能由一个线程写入，后32位由另一个不同的线程写入），不过加上volatile修饰后的long 和double也具有原子性。

注意：volatile关注可见性，而与原子性没有关系。volatile关注点在于从工作内存刷新回主内存，而原子操作关注的是否不被打断。原子和同步目的都是让不同线程可以**安全地访问**共享变量的两种处理方式，避免造成内存一致性错误。

我是葛一凡，希望对你有帮助。
![微信公众号](/img/qrcode.jpg "微信公众号")

## 参考
1. [聊聊并发（一）深入分析Volatile的实现原理](http://ifeve.com/volatile/)
2. [聊聊并发（五）——原子操作的实现原理](http://www.infoq.com/cn/articles/atomic-operation)
3. [Java Language Specification](http://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html)
4. [Java Volatile 关键字详解](http://lucumt.info/posts/java-volatile-keyword/)
5. [从缓存行出发理解volatile变量、伪共享False sharing、disruptor](http://www.cnblogs.com/pingyuyue/archive/2012/02/20/2360596.html)
6. [Java并发编程：volatile关键字解析](http://www.cnblogs.com/dolphin0520/p/3920373.html)
7. [Java 编程要点之并发（Concurrency）详解](https://yq.aliyun.com/articles/47316?spm=5176.100240.searchblog.28.qOvfNz)
8. [Java 并发编程(1): Java 内存模型(JMM)](http://www.jianshu.com/p/76a3648f0a9f)
