---
title: Spring Boot改成war包启动
date: 2020-05-24 19:00:25
updated: 2020-05-24 19:03:25
tags: 
- java
- Spring Boot
---

###概要

出于对Tomcat的信任，在Spring Boot可以直接用jar启动的情况下，还是决定改成war包启动。虽然都能配置jvm参数，都要日志输出，可以需要替换一些静态页面的时候war包可以直接替换，无需重新启动服务。

<!--more-->

### 步骤

1. 首先配置pom，去掉内置的Tomcat，改成直接引用starter-tomcat包，并设置scope为provided。从而保证打包的时候不会打入多余的tomcat相关的包，又能在本地直接通过main运行项目。

   ```xml
   <!-- 去掉内置tomcat。 -->
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web</artifactId>
     <exclusions>
       <exclusion>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-tomcat</artifactId>
       </exclusion>
     </exclusions>
   </dependency>
   
   <!-- 改成直接引用并设置socpe范围为provided，表明该包只在编译和测试的时候用。 -->
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-tomcat</artifactId>
     <scope>provided</scope>
   </dependency>
   ```

2. 增加一个初始化类继承SpringBootServletInitializer方法，重写configure方法，并通过sources方法把Spring Boot项目启动类传进去。

   ```java
   public class ServletInitializer extends SpringBootServletInitializer {
       @Override
       protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
           return application.sources(Application.class);
       }
   }
   ```

3. 到此配置完成，我们通过maven打成的war包就可以直接在tomcat中运行了。而我们实现SpringBootServletInitializer就相当于配置了web.xml。

### 总结

jar包改成war包是非常的容易，两种方式都有优劣，而到底使用哪种部署方式这就需要根据实际情况来判断了。

