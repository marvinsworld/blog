title: Python安装
date: 2016-04-25 15:07:39
categories: Begin Python
tags: [python,安装]
---
<img src="/img/install.png" width="250" height="250" class="img-topic" />
从接触python到现在有5年时间了,一直在研究爬虫方面。喜欢python的便捷,喜欢python的优雅,喜欢python的简单。
<!--more-->

# 简单

实现一个读写文件功能,如果用java,算上各种包装类,异常捕获,没20多行下不来；而python两行搞定。
```python
f=open('f.txt','w') 
f.write('aaa')
```

# 安装

新手建议先在windows下安装,因为mac和linux系统会自带python2.x,上手相对困难一些,不过更希望更多的开发者从windows上迁移到mac和linux上,个中原因,你懂的。

# Windows安装

推荐使用python3.x,从[官网](https://www.python.org/downloads/windows/),下载相应的系统版本。一路下一步即可。

安装后,需要设置变量。右击我的电脑--属性--高级系统设置--环境变量--系统变量,找到PATH。假如你的安装版本为3.5.1,安装目录为D:\ProgramSoft\Python35,在最前面加入如下片段,注意分号
```python
D:\ProgramSoft\Python35\Scripts\;D:\ProgramSoft\Python35\;
```
检测是否安装成功。调起cmd命令窗口。
```python
C:\Users\dmall>python -V
Python 3.5.1
```

# 开始

定义一个方法,
```python
def helloWorld()
	pass

print('Hello world')
```

# 不足

python流行的有两个版本2.7和python3,两个版本差异很大,这跟初学者带来了很大的不便,网上很多都是python2。如果新手建议使用python3,毕竟代表更先进的生产力。
