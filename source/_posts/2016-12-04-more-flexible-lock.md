title: 更加灵活的Lock
date: 2016-12-04 22:50:32
categories: Java并发演进之路
tags: [显示锁,中断,灵活]
photos:
    - /img/lock.png
---
Lock是一把相对于更加灵活的锁。灵活之处在于提供了超时、可中断的获取锁方式，把这种自由度交给用户控制；与synchronized不同，Java内置锁就像一个大包大揽的家长，把获取、释放锁都自己处理了。依然保持上文中的观点：如果仅仅是为了实现互斥，而不需要使用基于Lock的附加属性（中断、条件等），推荐优先使用synchronized。
<!--more-->

## 锁目的
> A lock is a tool for controlling access to a shared resource by multiple threads. Commonly, a lock provides exclusive access to a shared resource: only one thread at a time can acquire the lock and all access to the shared resource requires that the lock be acquired first.

锁是一种用于控制多个线程访问共享资源的工具。通常，锁提供了对共享资源的独占访问：同一时间只允许一个线程获取锁，在获取锁之后，才能访问共享资源（源自官方API文档）。锁的目的在于保护资源，为防止多个线程同时操作同一个资源，设计了Lock的方式，只有拿到Lock这个凭证，才有权操作资源。

Lock的设计本身就有乐观的含义。Lock的核心思路是：多个线程在同一时间访问同一资源的几率不大，在操作资源时一般不会出现争抢。因此首先尝试去获取锁，如果发现锁已经被占用，则使用自旋的方式重试（而不是使用休眠的方式等待锁的释放），直到获取锁。在资源争抢不严重的情况下，Lock效果更好，虽然会有CPU空转的情况，但是比休眠唤醒的成本要低很多。

## CAS原理
CAS是Compare and Swap，是CPU级别的指令，Lock是通过CAS这个原子指令获取锁标识和释放锁标识。CAS通过调用JNI代码实现，例如sun.misc.Unsafe类的compareAndSwapInt()方法的native实现，可以在OpenJDK的源代码hotspot\src\share\vm\prims\unsafe.cpp中找到：

```java
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END
```

Atomic::cmpxchg的实现不同的操作系统有所不同，以linux x86为例，源码位置在hotspot/src/os_cpu/linux_x86/vm/atomic_linux_x86.inline.hpp
```java
inline jint     Atomic::cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value) {
  int mp = os::is_MP();
  __asm__ volatile (LOCK_IF_MP(%4) "cmpxchgl %1,(%3)"
                    : "=a" (exchange_value)
                    : "r" (exchange_value), "a" (compare_value), "r" (dest), "r" (mp)
                    : "cc", "memory");
  return exchange_value;
}
```

对于LOCK前缀在Intel® 64 and IA-32 Architectures Software Developer’s Manual中详细的说明：在8.1.4中，
> For the Intel486 and Pentium processors, the LOCK# signal is always asserted on the bus during a LOCK operation, even if the area of memory being locked is cached in the processor.
> For the P6 and more recent processor families, if the area of memory being locked during a LOCK operation is cached in the processor that is performing the LOCK operation as write-back memory and is completely contained in a cache line, the processor may not assert the LOCK# signal on the bus. Instead, it will modify the memory loca- tion internally and allow it’s cache coherency mechanism to ensure that the operation is carried out atomically. This operation is called “cache locking.” The cache coherency mechanism automatically prevents two or more proces- sors that have cached the same area of memory from simultaneously modifying data in that area.

1. LOCKQ前缀会锁定总线（优化后为锁定缓冲行），使其他处理器无法读写该指令访问的区域，保证指令执行的原子性。
2. 禁止该指令与之前和之后的读和写指令重排序。
3. 把写缓冲区中的所有数据刷新到内存中。（2，3还未从手册中找到出处）

## Acquire和Release
通过加锁和解锁构成一个临界区，不用担心并发的冲突，保证对临界区的串行访问。Acquire和Release是一个整体，不能割裂使用。Acquire语义保证所有内存的读写操作必须在Acquire之后执行；Release语义保证所有的内存的读写操作必须在Release之前执行。

码字辛苦，痛并快乐着；理解可能存在偏差，字字斟酌推敲；抵制抄袭，践行原创技术之路。如能帮助读者一二，实为荣幸，我是葛一凡。
![微信公众号](/img/qrcode.jpg "微信公众号")

## 参考
1. [官方API文档](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/locks/Lock.html)
2. [深入理解Java内存模型（五）——锁](http://ifeve.com/java-memory-model-5/)
3. [并发编程系列之一：锁的意义](http://hedengcheng.com/?p=803)
4. Intel® 64 and IA-32 Architectures Software Developer’s Manual
5. [java中锁的探究（4）——乐观锁和悲观锁](http://www.jianshu.com/p/59ddb7002b30)
6. [Jdk1.6 JUC源码解析(1)-atomic-AtomicXXX](http://brokendreams.iteye.com/blog/2250109)
7. [acquire and release semantics in mutex的理解](https://www.zhihu.com/question/26588157)