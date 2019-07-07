title: AQS漫谈
date: 2017-01-21 20:00:01
categories: Java并发演进之路
tags: [AQS,基础,自旋,获取,释放]
photos:
	- /img/aqs-puzzle-piece.png
---
AbstractQueuedSynchronizer简称AQS，字面上解释为抽象队列同步器。AQS是java.util.concurrent.locks的核心，也是Java并发包的基础，就像拼图游戏中的最后一片拼块。
<!--more-->

本文从AQS的组件Condition和LockSupport入手，一步一步地去拨开AQS的外衣，在深入的过程中会把在学习AQS过程中的疑问一一解答，也许你同样有的这样的疑问，包括以下内容：
- LockSupport
- Condition
- tryAcquire
- acquireQueued

## LockSupport
> Basic thread blocking primitives for creating locks and other synchronization classes.

LockSupport在JDK源码中描述为：构建锁和其他同步类的基本**线程阻塞**原语。既然是原语，那么LockSupport提供了哪些功能呢？

## 阻塞的语义
阻塞是线程在某个节点由于某种原因而无法继续串行的方式执行。造成阻塞的原因有很多：等待获取一个已经被其他线程持有的排他锁、等待某一操作结束、等待一个时间段。线程阻塞后，通常会被挂起，此时会处于BLOCKED、WAITING、TIMED_WAITING。
