title: 更加灵活的锁
date: 2016-12-04 22:50:32
categories: Java并发演进之路
tags: [显示锁,中断,灵活]
---
<img src="/img/lock.png" width="300" class="img-topic" />
synchronized是一把互斥锁，但java封装过于封闭，以至于只要加上这个锁，就相当于把主动权交出去了，再也不受自己控制，直到解锁。这样有简单的好处，但不灵活，于是更加灵活的锁在Java 5.0后加入。
<!--more-->
