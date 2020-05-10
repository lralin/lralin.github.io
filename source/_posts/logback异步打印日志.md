---
title: logback异步打印日志
date: 2020/04/19 20:50:25
updated: 2020/04/19 20:50:25
tags:
- logback
- 日志

---

##### 简介

默认的logback是同步打印日志的，这就造成如果打印很大的内容的话，会影响程序的处理时间。本来一个请求1s就返回了，最终因为日志打印太久，增加了2s，大大影响了用户体验。于是就有了异步日志。

<!--more-->

##### 同步日志配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <contextName>logback</contextName>

    <!-- 日志存放路径配置 -->
    <property name="logDir" value="logs/%d{yyyyMMdd}"/>

    <!--输出到控制台-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </layout>
    </appender>

    <!-- 按照线程打印日志-API -->
    <appender name="SIFT" class="ch.qos.logback.classic.sift.SiftingAppender">
        <!--discriminator鉴别器，根据uuid这个key对应的value鉴别日志事件，然后委托给具体appender写日志-->
        <discriminator>
            <key>uniqueId</key>
            <defaultValue>sys</defaultValue>
        </discriminator>
        <sift>
            <!--具体的写日志appender，每一个uuid创建一个文件-->
            <appender name="File-${uniqueId}" class="ch.qos.logback.core.rolling.RollingFileAppender">
                <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                    <fileNamePattern>${logDir}/${uniqueId}.trc</fileNamePattern>
                </rollingPolicy>
                <layout class="ch.qos.logback.classic.PatternLayout">
                    <pattern>%d{yyyyMMdd HH:mm:ss.SSS} %-5level %msg%n</pattern>
                </layout>
            </appender>
        </sift>
    </appender>

    <root level="INFO">
        <appender-ref ref="console"/>
        <appender-ref ref="SIFT"/>
    </root>
</configuration>
```

同步打印配置如上，配置了控制台输出和按文件输出。

##### 异步日志配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <contextName>logback</contextName>

    <!-- 日志存放路径配置 -->
    <property name="logDir" value="logs/%d{yyyyMMdd}"/>

    <!--输出到控制台-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </layout>
    </appender>

    <!-- 按照线程打印日志-API -->
    <appender name="SIFT" class="ch.qos.logback.classic.sift.SiftingAppender">
        <!--discriminator鉴别器，根据uuid这个key对应的value鉴别日志事件，然后委托给具体appender写日志-->
        <discriminator>
            <key>uniqueId</key>
            <defaultValue>sys</defaultValue>
        </discriminator>
        <sift>
            <!--具体的写日志appender，每一个uuid创建一个文件-->
            <appender name="File-${uniqueId}" class="ch.qos.logback.core.rolling.RollingFileAppender">
                <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                    <fileNamePattern>${logDir}/${uniqueId}.trc</fileNamePattern>
                </rollingPolicy>
                <layout class="ch.qos.logback.classic.PatternLayout">
                    <pattern>%d{yyyyMMdd HH:mm:ss.SSS} %-5level %msg%n</pattern>
                </layout>
            </appender>
        </sift>
    </appender>

  	<!-- 异步输出 -->
    <appender name ="ASYNC" class= "ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
        <discardingThreshold >0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>512</queueSize>
        <!-- 添加附加的appender,最多只能添加一个 -->
        <appender-ref ref ="SIFT"/>
    </appender>
  
    <root level="INFO">
   <!-- <appender-ref ref="CONSOLE"/>-->
      	<appender-ref ref ="ASYNC"/>
    </root>
</configuration>
```

生产日志可以去掉控制台输出，只保留异步日志输出。

