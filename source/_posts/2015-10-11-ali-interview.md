title: 面试感悟
date: 2015-10-11 15:21:28
tags: [面试,感悟,整体,解决思路]
categories: 感悟
photos:
	- /img/ali_think.jpg
---
优秀的面试官会对面试问题进行引导和扩展，并据此分析面试者的分析问题和解决问题的方式和思路，从而侧面了解面试者的优缺点。不合格的面试官的面试场景通常是这样的：

面试官：请你讲讲SpringMVC的DispatcherServlet的原理（一上来有些深度）

面试者：不太清楚（茫然）

面试官：……

面试官：下一话题（好吧，没法继续了）
<!--more-->
而优秀的面试官会从其他方面，比如，DispatcherServlet是怎么使用的，如果让你自己来设计DispatcherServlet，你会怎么如何设计？或者从url路径和方法的映射关系入手谈谈，再或者数据是如何渲染。

# 面试的角度

曾经见过一篇面试HashMap的题目，给我的感触是这道题目逐层深入，从算法，并发和思维层面都进行了考量，对我思考其他问题也提供了一些思路，真是受益匪浅。问题是这样的：

1. *什么是HashMap？你为什么用到它？*
2. *你知道HashMap的get()方法的工作原理吗？*
3. *当两个对象的hashcode相同会发生什么？*
4. *如果两个键的hashcode相同，你如何获取值对象？*

接下来深入了

1. *如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？*
2. *你了解重新调整HashMap大小存在什么问题吗？*

再往下我们就可以说到ConcurrentHashMap了……而如果只问HashMap和HashTable的区别真是太小儿科了

# 面试的总结

其实我想说说面试的感触，不是想鄙视一下不合格的面试官，也不是想表现一下自己多么多么的牛逼，只是想通过一些自己的切身体会，希望面试不是面试官鄙视面试者，面试官强势压倒面试者，而是希望面试官重视面试技巧，面试者思考面试内容，从而使面试打开一种双赢的局面。

