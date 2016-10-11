title: 为什么我们要考虑并发
date: 2016-10-10 06:58:03
categories: Java并发演进之路
tags: [并发,上下文切换,效率，线程数]
---
<img src="/img/cpu.png" width="300" height="300" class="img-topic" />
“很显然嘛，并发可以提升效率”

我最初开始学习并发时，觉得原因显而易见，多线程执行肯定比单线程快，就像多个人一块干活要比一个人快。不过事实远比想象的要复杂，让我们把视角缩小到CPU级别，细致地观察一下多线程并发时，CPU在做什么。
<!--more-->

## CPU和I/O耗时
CPU计算操作所需时间一般在纳秒级别，而I/O操作（包括内存、磁盘）所需时间差距很大，例如CU一级缓存的相应时间在纳秒级别，而磁盘读取文件的响应时间在毫秒级别，前者较后者相比快5-6个数量级。CPU就像疾如闪电的快银（漫威动画），而磁盘是一个慢吞吞的蜗牛，这两位在一起工作，而且CPU必须I/O操作完成后才进行后续的操作，这对于CPU是一种煎熬，更是一种资源的巨大浪费。

智慧的操作系统设计者已经想到这点了，适时引入了CPU上下文切换，提高资源的利用率，降低整体的相应时间，避免CPU这种宝贵且昂贵（对比一下你组装电脑时CPU的价格和内存条的价格）的资源浪费。

## CPU上下文切换
在多任务处理系统中往往会有同时进行多个任务，然而一个CPU核心同一时间只能处理一个任务。操作系统设计者使用了时间片轮转的方式。下图浅蓝色是CPU的时间线，不同颜色的小方块是不同的任务，沿着时间线，CPU在不同的任务间切换。
![CPU上下文切换](/img/cpu-context-switch.png "CPU上下文切换")
进一步缩小视角，CPU在任务间切换时，先保存（save）当前任务的状态，然后恢复（restore）下一任务的状态，再执行下一任务。下图引自并发编程网。
![CPU上下文切换细节](/img/context-switch.png "CPU上下文切换细节")

## 计算密集型和I/O密集型
计算密集型是CPU在进行大量的计算，任务没有太多的I/O操作，使用多线程无法使性能得到过多的提升，相反，过多的线程数，由于CPU会发生过多次数的上下文切换，反而会浪费CPU时间，降低效率。I/O密集型是任务在做大量的读写操作，站在CPU的角度来看，I/O一直是阻塞状态，这时可以使用多线程切换到其他任务，提升整体执行效率。很荣幸，一般的任务都是既有CPU计算，又有I/O，当然还有网络传输，最慢的那块就是整个任务的瓶颈，这就是木桶原理，有一块短板，就会拖慢整个过程。那么CPU在这大把的空闲时间内该如何做呢？答案一目了然：通过并发编程的方式提升性能，降低相应时间。

## 线程的数目
既然已经确定使用多线程，那么线程的数目是多少呢？总不能无来由的想一个吧？首先确定任务是CPU密集型（加解密，排序），还是I/O密集型（文件读写，网络）。
对于CPU密集型，为了减少CPU上下文切换的频率，最佳线程数=CPU核心数，具体的线程数需要结合实际项目，依据实际监控数据调整得到。
对于I/O密集型，CPU大多数情况下是空闲的，最佳线程数可以参考以下公式：** 最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 ）\* CPU核心数**
查看CPU核心数：`cat /proc/cpuinfo |grep "processor"|wc -l`
这只是一个估算，在实际生产环境中，需要结合监控数据，硬件环境调整优化，确定系统的最佳点。如何获取线程等待时间和线程CPU时间，可以参考这段代码：
```java
import static org.junit.Assert.assertTrue;

import java.lang.management.ManagementFactory;
import java.lang.management.ThreadInfo;
import java.lang.management.ThreadMXBean;

import org.junit.Test;

public class ThreadStatsTest {

	@Test
	public void testThreadStats() throws Exception {
		ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
		assertTrue(threadMXBean.isThreadCpuTimeSupported());
		assertTrue(threadMXBean.isCurrentThreadCpuTimeSupported());

		threadMXBean.setThreadContentionMonitoringEnabled(true);
		threadMXBean.setThreadCpuTimeEnabled(true);
		assertTrue(threadMXBean.isThreadCpuTimeEnabled());

		ThreadInfo[] threadInfo = threadMXBean.getThreadInfo(threadMXBean.getAllThreadIds());
		for (ThreadInfo threadInfo2 : threadInfo) {
			long blockedTime = threadInfo2.getBlockedTime();
			long waitedTime = threadInfo2.getWaitedTime();
			long cpuTime = threadMXBean.getThreadCpuTime(threadInfo2.getThreadId());
			long userTime = threadMXBean.getThreadUserTime(threadInfo2.getThreadId());

			String msg = String.format(
					"%s: %d ns cpu time, %d ns user time, blocked for %d ms, waited %d ms",
					threadInfo2.getThreadName(), cpuTime, userTime, blockedTime, waitedTime);
			System.out.println(msg);
		}
	}
}
```

## 参考
1. [关于并发的思考](http://ningandjiao.iteye.com/blog/2184456)
2. [如何合理地估算线程池大小？](http://ifeve.com/how-to-calculate-threadpool-size/)
3. [从Java视角理解系统结构(一)CPU上下文切换](http://ifeve.com/what-is-context-switching/)
4. [什么是上下文切换](http://ifeve.com/java-context-switch/)
5. [How can I Monitor cpu usage per thread of a java application in a linux multiprocessor environment?](http://stackoverflow.com/questions/1680865/how-can-i-monitor-cpu-usage-per-thread-of-a-java-application-in-a-linux-multipro)



