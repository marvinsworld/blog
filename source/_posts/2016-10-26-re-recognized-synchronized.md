title: 重新认识synchronized
date: 2016-10-26 18:21:30
categories: Java并发演进之路
tags: [锁,监视器]
---
<img src="/img/synchronized.png" width="400" class="img-topic" />
synchronized在JDK5之前一直被称为重量级锁，是一个较为鸡肋的设计，而在JDK6对synchronized内在机制进行了大量显著的优化，加入了CAS，轻量级锁和偏向锁的功能，性能上已经跟ReentrantLock相差无几，而且synchronized在使用上更加简单，不易出错（避免哲学家就餐问题造成的死锁），因此如果仅仅是为了实现互斥，而不需要使用基于Lock的附加属性（中断、条件等），推荐优先使用synchronized。
<!--more-->

## synchronized用法
synchronized有同步方法和同步语句块两种形式，分锁定对象和作用域两个维度，当锁定的对象进入作用域时，互斥才会生效。**一个线程访问拥有相同锁定对象时，进入作用域时必须以串行方式进行**。修饰的对象有：修饰普通方法（又称为同步方法），修饰静态方法，修饰对象实例，修饰class literals（类名称字面常量）。

### 修饰普通方法
又称同步方法，锁定的是调用这个同步方法对象。一个线程访问实例对象中的synchronized代码块时，其他试图访问**该对象synchronized修饰的区域**的线程将阻塞。同一个对象共享一把锁，同一时间只能有一个synchronized区域可以获得锁，可以称为**对象锁**。需要注意两点：（1）作用只会在同一个实例上，（2）对象中的所有synchronized修饰的区域。例如：对于fooA对象，线程t1，t2访问methodA是互斥的，线程t1，t2分别访问methodA，methodB同样是互斥的；当访问methodA时，methodB 不可以访问，methodC可以访问。
```java
public class Foo implements Runnable {
    public static synchronized void methodA() {
        //TODO
    }

    public static synchronized void methodB() {
        //TODO
    }

    public void methodC() {
        //TODO
    }

    public static void main(String[] args) {
        Foo fooA = new Foo();
        Foo fooB = new Foo();

        Thread t1 = new Thread(fooA, "t1");
        Thread t2 = new Thread(fooB, "t2");
        t1.start();
        t2.start();
    }
}
```

### 修饰静态方法
锁定是静态方法所属的类，此时该类共用一把锁，可以称为**类锁**。一个线程访问synchronized静态方法时，其他试图访问**该类synchronized修饰的区域**的线程都将阻塞，而非synchronized修饰的区域不会阻塞。修饰静态方法是修饰普通方法的变种版，前者锁定该类（跟对象没关系），后者指锁定当前对象。例如：线程t1，t2分别访问methodA，methodB同样是互斥的。

### 同步语句块
修饰对象实例的情况与修饰普通方法一致，区别在于只有线程进入同步语句块时，才是互斥的。而修饰class literals（类名称字面常量）与修饰静态方法一致。例如：methodD锁定的是当前对象，而methodE中的synchronized锁定的是Foo类的所有对象。
```java
public class Foo {
    public void methodD() {
        synchronized (this) {
            //TODO
        }
    }

    public void methodE() {
        synchronized (Foo.class) {
            //TODO
        }
    }
}
```

### 类锁和对象锁互斥吗
synchronized修饰静态方法（类锁）和修饰普通方法（对象锁）之间没有竞争关系，不是互斥的。前者锁定的是Class，后者锁定的Instance，二者没有关系，不会相互影响。**对于实例同步方法，锁是当前实例对象。对于静态同步方法，锁是当前对象的Class对象。**

## synchronized机制
synchronized是Java为线程间通信提供的一种方式，synchronized的这种同步机制是互斥锁机制。实现互斥的方式有：临界区（Critical Section），互斥量（Mutex），信号量（Semaphores）和事件（Event）。

synchronized是采用临界区的方式实现互斥的，通过对多线程的串行化来访问公共资源或一段代码，保证同一时刻只有一个线程能访问临界区内共享数据。如果多个线程试图同时访问临界区，其中一个线程抢占临界区后，其他试图访问该临界区的线程将会挂起，直到进入临界区的线程。举个栗子，会议室资源相对紧张，有多个团队都需要使用会议室，这时就看哪个团队的迅速了。当这个团队抢占到会议室后，就会挂起“正在会议中”的提示牌，其他需要使用会议室的人看到会等待，直到当前使用会议室的团队使用完，移走“正在会议中”的提示牌。

synchronized同时保证了可见性。互斥锁释放后，确保一个线程修改的对象状态，对其他线程是可见的，即在随后获取锁的线程中保证获得最新共享资源的状态。


## 参考
1. [Java synchronized关键字解析](http://sawyersun.github.io/2016/10/18/Java-synchronized/)
2. [临界区，互斥量，信号量，事件的区别](http://www.cnblogs.com/mr-m/p/3549919.html)

