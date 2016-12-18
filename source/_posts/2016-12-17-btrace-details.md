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

