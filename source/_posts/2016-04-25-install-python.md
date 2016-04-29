title: Python安装
date: 2016-04-25 15:07:39
categories: Begin Python
tags: [python,安装]
---
<img src="/img/install.png" width="250" height="250" class="img-topic" />
Python是诗一般富有美感的语言。从设计上可以体现出python在刻意的追求这种美感，在开发中，给人以丝绸般顺滑的感觉（有点像XX的广告，Python是不是考虑让我做代言人了呢？）

Python是优雅的语言。一位名人说过这样一句话：追求优雅生活从Python开始。（我说的）从容优雅，大方得体。不像Java一会强制你考虑强制类型转换，一会强制考虑异常捕获。Python给人一种宽松愉悦的氛围，让人沉浸其中。
<!--more-->

# Python的优雅

Python没有大括号（｝）的概念，函数的定义，循环，分支等的区分是由Tab和换行完成的。Python有着严格的缩进要求，所以Python统一了排版方式，在阅读他人代码时，不用担心出现千人千面的程序排版。

# Python的简单

如何你有一些其他语言的基础，就会发现Python上手非常简单。
实现一个读写文件功能，如果用java，算上各种包装类，异常捕获，没20多行下不来；而python两行搞定。
```python
f=open('f.txt','w') 
f.write('aaa')
```
爬去百度首页，用几行代码完成完成了，so easy！
```python
reponse = urllib.request.urlopen('http://wwww.baidu.com')
data = reponse.read()
```

# Windows上安装

新手建议先在windows下安装,因为mac和linux系统会自带python2.x,上手相对困难一些,不过更希望更多的开发者从windows上迁移到mac和linux上,个中原因,你懂的。

推荐使用python3.x,从[官网](https://www.python.org/downloads/windows/),下载相应的系统版本。一路下一步即可。

安装后,需要设置变量。右击我的电脑--属性--高级系统设置--环境变量--系统变量,找到PATH。假如你的安装版本为3.5.1,安装目录为D:\ProgramSoft\Python35,在最前面加入如下片段,注意分号
```python
D:\ProgramSoft\Python35\Scripts\;D:\ProgramSoft\Python35\;
```
检测是否安装成功。调起cmd命令窗口。
```python
C:\Users\Eason>python -V
Python 3.5.1
```

# Mac上安装

暂时

# Linux上安装

暂时略

# The Red Pill Or The Blue

python流行的有两个版本2.7和python3,两个版本差异很大,这跟初学者带来了很大的不便,网上很多都是python2。如果新手建议使用python3,毕竟代表更先进的生产力。

Python在2.x和3.x上一直被人视为最大的败笔。时代在进步，历史在召唤，我会坚定不移的沿着Python3的这条道路上走下去，你呢？
