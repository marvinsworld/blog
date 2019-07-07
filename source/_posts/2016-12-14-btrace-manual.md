title: Btrace入门
date: 2016-12-14 14:23:04
categories: 实战性能优化
tags: [监控,利器,堆栈,Btrace]
photos:
    - /img/btrace-diagnosis.jpg
---
在生产环境中可能经常遇到各种问题，排查定位问题需要当时程序运行的更多信息，如方法参数、返回值、全局变量、堆栈信息等，不可能在开发的时候就把程序中所有的运行细节都打印到日志上，通过的做法是修改代码，增加日志信息的打印，部署到生产环境再观察。但是这种方式的问题很明显：一方面将增大定位问题的成本和周期，对于紧急问题无法做到及时响应；另一方面重新部署后环境可能已被破坏，很难重新问题的场景。
<!--more-->

Btrace就是为解决这样的情形出现的。Btrace是一款动态查看程序运行细节的工具。无需修改应用代码（事实上它修改了字节码），程序无需任何多余依赖，无需重启。怎么说呢，Btrace就是线上的debug模式。Btrace官方已经迁移到github上，详见[官网](https://github.com/btraceio/btrace)

## 能解决什么问题
Btrace被称为线上排查问题的神器，那么它适合在哪些场景下使用呢？
1. 监控某个方法的执行时长
2. 抛出异常时查看方法入参、对象属性、相应的分支流程
3. 某个方法执行的堆栈信息
4. 哪些对象向Map或List 不断地创建对象
5. 频繁的YoungGC时，如何快速定位哪个类在不断地创建对象

当然，第一种场景下最优的方式是性能监控系统（监控到方法级别），而第二种场景最优方式是捕获可能存在的异常并通过日志记录引发异常的具体内容。或者是第一，二种场景下的补救措施：日志记录的粒度不够或者日志级别过高。Btrace最厉害的地方在于，你怀疑某个类在频繁的创建对象，就可以使用Btrace监控，或者从一个较大的范围一点点的缩小范围，最后定位问题的根源。

## 局限
Btrace不是万能的，Btrace是通过HotSpot虚拟机的HotSwap技术动态插入原本不存在的调试代码，其是基于了JDK 6的Instumentation来实现的。为了避免对线上运行程序的性能影响过大，Btrace在编写监控代码方面做了很多限制，比如只能调用BTraceUtils 里的一系列方法和脚本里定义的static方法，不允许其他调用任何类的任何方法。

需要重点注意的是：Btrace植入后的代码，在下次启动前，是不会清除的，即使BTrace退出了，被监控方法每次执行时都会再执行一遍。

## 基本使用
Btrace最新的版本是1.3.8，下载地址是[https://github.com/btraceio/btrace/releases/tag/v1.3.8.3-1](https://github.com/btraceio/btrace/releases/tag/v1.3.8.3-1)，解压至指定目录，设置BTRACE_HOME环境变量
```java
vim ~/.bash_profile
输入
export BTRACE_HOME=/usr/local/btrace-bin-1.3.8.3
export PATH=$BTRACE_HOME/bin:$PATH
生效
source ~/.bash_profile
```

举个栗子，一个加法计算器，输入两个数字球和：
```java
package com.ewan.test.btrace;

import java.util.Scanner;

public class Calculator {
    public int add(int a, int b) {
        int c = a + b;
        return c;
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        Calculator roboot = new Calculator();
        while (scanner.hasNext()) {
            int a = scanner.nextInt();
            int b = scanner.nextInt();
            int c = roboot.add(a, b);
            System.out.printf(String.format("%d + %d = %d", a, b, c));
        }
    }
}
```
监控脚本：
```java
package com.ewan.test.btrace;

import com.sun.btrace.annotations.*;
import static com.sun.btrace.BTraceUtils.*;

@BTrace
public class BtraceScript {
    @OnMethod(clazz = "com.ewan.test.btrace.Calculator", method = "add", location = @Location(Kind.RETURN))
    public static void returnFunc(@Self Object self, @Return int result, @Duration long time, int a, int b) {
        println("returnFunc: =======================");
        jstack();
        println(strcat(strcat(strcat("a:", str(a)), ",b:"), str(b)));
        println(strcat(strcat(strcat("result:", str(result)), ",time:"), str(time)));
        println();
    }
}
```

1. 运行加法计算器：`java com.ewan.test.btrace.Calculator`
2. 查看进程pid：`jps | grep Calculator | awk '{print $1}'`
3. 运行监控脚本（运行应该在BtraceScript.java同级目录，不用关注包名）：`btrace  BtraceScript.java`
4. 演示结果
```java
returnFunc: =======================
com.ewan.test.btrace.Calculator.add(Calculator.java:15)
com.ewan.test.btrace.Calculator.main(Calculator.java:24)
a:3,b:4
result:7,time:1000
```
5. 解释运行结果
入参a，b分别是3，4，输出结果是7，执行市场是1000ns

我是葛一凡，希望对你有帮助。
![微信公众号](/img/qrcode.jpg "微信公众号")

## 参考
1. [官网](https://github.com/btraceio/btrace)
2. [BTrace简介及使用](http://blog.csdn.net/wildandfly/article/details/21107661)
3. [Java BTrace实战(1)--BTrace的入门和使用](http://www.cnblogs.com/mumuxinfei/p/3944823.html)
