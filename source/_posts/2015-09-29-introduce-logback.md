title: Logback介绍
date: 2015-09-29 14:52:49
tags: [root,logger]
categories: 基础
photos:
	- /img/logback_logo.jpg
---
经常使用logbak,却对配置文件一知半解,今天好好研究了一下.
<!--more-->
# Logback的层级
![Alt text](/img/logback_basic.png "logback层级关系")
Logback主要分三个层级,分别是`<appender>`,`<logger>`和`<root>`.
* `<appender>`中定义了日志文件输入的路径(`<file>`),滚动记录文件的规则(`<RollingFileAppender>`),以及输出格式的规则(`<encoder>`)
* `<logger>`针对某个类或者某个包下日志的输入规则
* `<root>`针对所有类的日志输出规则

一个形象的比喻,`<root>`和`<logger>`是父子关系,`<appender>`是一支笔,定义了线条的规则,而`<root>`,`<logger>`都要使用这支笔才能画图.`<logger>`可以选择按照`<root>`制作的毛笔画图,也可以选择重新制作的钢笔画图.

# Logback的基础知识
在`<logger>`的appender会根据属性additivity决定是否将打印信息向上传递到`<root>`的appender.比如:logback会把com.marvinsworld.HelloWorld类中info及以上的日志根据名字为INFO_LOG的appender中定义的格式输出.
```xml
<logger name="com.marvinsworld.HelloWorld" level="info" additivity="false">
    <appender-ref ref="INFO_LOG"/>
</logger>
```

如果不想输出指定的包或者类下面的日志,可以这么做:
```xml
<logger name="com.marvinsworld.HelloWorld" level="info" additivity="false" />
```

如果每天生成的日志文件太大,可以同时结合日期和数字来生成日志文件:简单说明一下,此代码是需要放在`<appender>`中的,第一次生成的文件时info.log,超过20M后会进行压缩后存储为例如info.2015-10-25.0.log.gz,其中0为序号,会自增.
```xml
<file>/web/logs/info.log</file>
<!-- rollover daily -->
<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
    <fileNamePattern>/web/logs/info.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
    <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
    	<!-- or whenever the file size reaches 20MB -->
        <maxFileSize>20MB</maxFileSize>
    </timeBasedFileNamingAndTriggeringPolicy>
    <!-- keep 30 days' worth of history -->
    <MaxHistory>30</MaxHistory>
</rollingPolicy>
```