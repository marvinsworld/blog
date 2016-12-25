title: 深入Btrace
date: 2016-12-17 20:51:55
categories: 实战性能优化
tags: [细节,利器,堆栈,Btrace]
---
<img src="/img/btrace_dig.png" width="300" class="img-topic" />
虽然Btrace很强大，但是在官方资料匮乏的情形下，很多功能都需要使用者去挖掘。推荐使用人如果有时间的话，研究官方一手资料或者源码，毕竟二手或多手资料与官方可能存在差异，本人也秉着大胆猜测，小心求证的原则，把已经证实的内容的细节列举一下，抛砖引玉。
<!--more-->

## 原理
> BTrace是基于动态字节码修改技术(Hotswap)来实现运行时java程序的跟踪和替换。大体的原理可以用下面的公式描述：Client(Java compile api + attach api) + Agent（脚本解析引擎 + ASM + JDK6 Instumentation） + Socket

![Java 线程监控模型](/img/btrace-jvmti.gif "Java 线程监控模型")

## 术语
**探测点（Probe Point）**：希望BTrace 跟踪脚本执行的**位置**和**时机**，比如进入某个类的某个方法时触发脚本的执行。

**跟踪动作/动作(Trace Actions or Actions)**：即触发后运行的Btrace 跟踪脚本。

**处理方法 (Action Methods)**：BTrace跟踪脚本需要放在一个类的静态方法中，这个静态方法称为处理方法。

## 关键词
`@OnMethod(①clazz = "com.ewan.test.btrace.Calculator", ②method = "add", ③location = @Location(④ Kind.RETURN))`

①处clazz表示跟踪的目标类，可以指定全限定类名，也可以是正则表达式。（例如: `"/com\\.ewan\\..+/"`，由于`.`是正则表达式中的特殊字符，如果想表达包层级分隔的含义，需要用`\\`来转义，`.+`表示的含义是至少有一个字符，因此这个简单的正则表达式的含义是com.ewan下所有子包中的所有类）

②处method表示目标类中的方法，组合①和②才是术语中的**探测点**的含义，这两处都可以使用正则表达式来跟踪多个类的多个方法。

③处location表示BTrace脚本触发的时机。

④ 处Kind定义触发的方式。在Kind中都已经定义好触发的时机，常用的有ENTRY（进入方法时，不写Location默认是ENTRY），RETURN（方法正常执行完成返回时触发），Throw（异常抛出），Catch（异常被捕获），Error（异常没被捕获而被抛出函数之外，主要用于对某些异常情况的跟踪）。

## 细节
1. ① clazz和② method对应的值只有两种形式，完全匹配和正则表达式。使用完全匹配时，不需要特殊处理；使用正则表达式时，需要把正则放在两个正斜杠`/`的中间，比如`"/com\\.ewan\\..+/"`。
2. 不要盲目的跟踪过多的类和方法，一方面不容易定位问题，另一方面性能会非常慢。不能为了解决一个问题，而引入了新的问题。
3. 特殊方法定位。构造函数使用`method="<init>"`
4. 跟踪接口：`clazz="+java.util.Map", method="put"`
5. 跟踪注解：`clazz="@javax.jws.WebService",method="@javax.jws.WebMethod"`

## 一些理解
BTrace的官方文档过于简单，虽然被誉为神器，不过很多技巧需要在实践过程中摸索总结。BTrace擅长的地方是通过问题呈现出来的现象（比如内存使用率居高不下、接口响应时间过长），结合排查问题经验，一步一步的缩小问题的怀疑范围，最终定位问题。

这里说说我对BTrace使用过程中浅显的理解，有理解不对的地方还请指正。

1. @TLS用于多个跟踪脚本之间的通信，通过定义 ThreadLocal的变量来共享，而很多文章中举的例子都是为查看某个方法的执行时间，Kind.RETURN和@Duration组合使用就很简单的实现了，不需要舍近求远。
2. Kind.CALL关注的探测点内部的逻辑，这与其他要区分清楚，另外还需要结合@Location内的clazz，method和跟踪脚本的变量参数一块使用，比较复杂，有时间详细说明一下。

我是葛一凡，希望对你有帮助。
![微信公众号](/img/qrcode.jpg "微信公众号")

## 参考
1. [BTrace原理浅析](http://www.rowkey.me/blog/2016/09/20/btrace/) 
2. [官网](https://github.com/btraceio/btrace)
3. [官方UserGuide1.2版](https://kenai.com/projects/btrace/pages/UserGuide)
4. [Btrace入门到熟练小工完全指南](http://calvin1978.blogcn.com/articles/btrace1.html)
5. [BTrace简介及使用](http://blog.csdn.net/wildandfly/article/details/21107661)
6. [Java BTrace实战(1)--BTrace的入门和使用](http://www.cnblogs.com/mumuxinfei/p/3944823.html)
7. [基于 JVMTI 实现 Java 线程的监控](https://www.ibm.com/developerworks/cn/java/j-lo-jvmti/)
8. [如何在生产环境使用Btrace进行调试](http://www.jianshu.com/p/dbb3a8b5c92f)