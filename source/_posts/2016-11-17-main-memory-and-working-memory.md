title: 主内存和工作内存
date: 2016-11-17 22:28:26
categories: Java并发演进之路
tags: [主内存,工作内存,共享变量]
photos:
	- /img/cpu-cache-memory.png
---
主内存（Main Memory）和工作内存（Working Memory）是为了形象理解缓存和CPU级别缓存而创造出来的一组概念。Java内存模型（JMM）中通过主内存和工作内存来屏蔽具体硬件和操作系统的差别，JMM规定主内存用于存储所有变量，每个线程还有自己的工作内存，线程的工作内存中保存了被该线程使用到的主内存变量的副本拷贝，线程对变量的所有操作（读取、赋值等）都必须在工作内存中进行，而不能直接读取主内存中的变量。
<!--more-->

## CPU与内存
大家都知道CPU的处理速度和内存的访问速度相差很大，比如Intel Sandy Bridge中，寄存器的一个处理周期小于1ns，而内存的访问速度大于在65ns，这就是“木桶原理”：一只水桶能装多少水取决于它最短的那块木板。除了提升内存的访问速度外（硬件成本高），CPU Cache的设计想通过提升读取数据的命中率，不需要从内存或磁盘上查找，来提升整体性能。设计Cache基于程序的局部性(Locality)
，即指程序在执行时呈现出局部性规律，即在一段时间内，整个程序的执行仅限于程序中的某一部分。相应地，执行所访问的存储空间也局限于某个内存区域。

CPU寄存器，L1、L2、L3cache，内存，磁盘，远程存储（网络），从左向右访问速度依次下降而存储容量依次提高，价格依次下降，这是存储层次的核心思想。类似印度游戏汉诺塔，从下往上依次增大。

## 缓存行
缓存行（Cache Line）是Cache访问的最小单元。CPU Cache与内存交换数据不是以字节为单位的，而是以块（Block）为单位，通常是32字节或64字节（根据操作系统的）。缓存行设计的目的是为了减少CPU Cache与内存交换数据的次数。 

## MESI协议
CPU Cache的设计虽然可以增加命中率，但是由于Cache的容量远远小于内存，因此难免会出现缓存不命中（Cache Miss）的情况。发生Cache Miss后，Cache会从下一层缓存中读取，如果当前缓存已满，会通过淘汰机制（例如LRU）覆盖被淘汰的数据块。

TODO

我是葛一凡，希望对你有帮助。
![微信公众号](/img/qrcode.jpg "微信公众号")

## 参考
1. [高性能服务端系列 -- 处理器篇](https://yq.aliyun.com/articles/8061)
2. [Java专家系列：CPU Cache与高性能编程](http://geek.csdn.net/news/detail/114619)
3. [【并发编程】CPU cache结构和缓存一致性（MESI协议）](http://blog.csdn.net/reliveIT/article/details/50450136)
4. [Java中的伪共享以及应对方案](https://yq.aliyun.com/articles/62865?utm_campaign=wenzhang&utm_medium=article&utm_source=QQ-qun&utm_content=m_7536)
5. [为什么程序员需要关心顺序一致性](http://www.parallellabs.com/2010/03/06/why-should-programmer-care-about-sequential-consistency-rather-than-cache-coherence/)
6. [关于CPU Cache -- 程序猿需要知道的那些事](http://cenalulu.github.io/linux/all-about-cpu-cache/)
7. [缓存一致性（Cache Coherency）入门](http://www.infoq.com/cn/articles/cache-coherency-primer)
8. [CPU Cache & False Sharing & 存储器层次结构](http://novoland.github.io/c%E5%92%8Cos/2014/07/26/CPU%20Cache%20%E4%B8%8E%20%E5%AD%98%E5%82%A8%E5%99%A8%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84.html)
9. [CPU缓存刷新的误解](http://ifeve.com/cpu-cache-flushing-fallacy-cn/)
10. [Java中的伪共享以及应对方案](https://yq.aliyun.com/articles/62865?utm_campaign=wenzhang&utm_medium=article&utm_source=QQ-qun&utm_content=m_7536)