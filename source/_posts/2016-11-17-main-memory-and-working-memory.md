title: 主内存和工作内存
date: 2016-11-17 22:28:26
categories: Java并发演进之路
tags: [主内存,工作内存,共享变量]
---
<img src="/img/cpu-cache-memory.png" width="400" class="img-topic" />
主内存（Main Memory）和工作内存（Working Memory）是为了形象理解缓存和CPU级别缓存而创造出来的一组概念。Java内存模型（JMM）中通过主内存和工作内存来屏蔽具体硬件和操作系统的差别，JMM规定主内存用于存储所有变量，每个线程还有自己的工作内存，线程的工作内存中保存了被该线程使用到的主内存变量的副本拷贝，线程对变量的所有操作（读取、赋值等）都必须在工作内存中进行，而不能直接读取主内存中的变量。
<!--more-->