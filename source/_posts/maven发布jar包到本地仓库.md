---
title: maven添加本地jar包到本地仓库
date: 2019/11/22 18:15:25
updated: 2019/11/22 18:25:25
tags:
- junit
- 测试
- 性能测试
---


### 简介
有些API接口有第三方的jar包，这些jar并不存在于中央仓库。当我们没有私服的情况下，我们又要用maven管理jar包。这就需要自己把jar包发布到本地私服，然后才可以直接在项目的pom文件里面引用。

<!--more-->

### 步骤

1. 创建lib文件夹。放入bcprov-jdk15on-153.jar包和install.bat、install.sh脚本

2. 编辑`install.bat`脚本

   ```shell
   call mvn install:install-file -DgroupId=com.bcprov -DartifactId=jdk15on -Dversion=1.5.3 -Dpackaging=jar -Dfile=bcprov-jdk15on-153.jar -DgeneratePom=true
   PAUSE
   #如果有源文件也可以使用下面这句话
   #call mvn install:install-file -DgroupId=org.jtester -DartifactId=jtester -Dversion=1.1.8 -Dpackaging=jar -DpomFile=jtester-1.1.8.pom -Dfile=jtester-1.1.8.jar -Dsources=jtester-1.1.8-sources.jar
   ```

3. 编辑`install.sh`脚本

   ```shell
   mvn install:install-file -DgroupId=com.bcprov -DartifactId=jdk15on -Dversion=1.5.3 -Dpackaging=jar -Dfile=bcprov-jdk15on-153.jar -DgeneratePom=true
   #mvn install:install-file -DgroupId=org.jtester -DartifactId=jtester -Dversion=1.1.8 -Dpackaging=jar -DpomFile=jtester-1.1.8.pom -Dfile=jtester-1.1.8.jar -Dsources=jtester-1.1.8-sources.jar
   ```

4. 在maven都已经配置环境变量的情况下，window执行`install.bat`脚本，linux执行`install.sh`

5. 项目pom.xml文件就中可以配置

   ```xml
   <dependency>
       <groupId>com.bcprov</groupId>
       <artifactId>jdk15on</artifactId>
       <version>1.5.3</version>
   </dependency>
   ```

   