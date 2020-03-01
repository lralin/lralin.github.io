---
title: idea插件Alibaba Cloud Toolkit
date: 2020-03-01 11:19:25
updated: 2020-03-01 11:19:25
tags: 
- idea
- java

---

### 前言

相信用过这个插件的人都说好，能快速的一键发布代码到开发环境，快速的登录到服务器，只需简单的配置，没有Jenkins那么复杂，自从用了cloud toolkit之后，发布开发环境的代码就不想用Jenkins了。

<!--more-->

### 发包

一般我们部署开发环境是先在本地打好包，然后通过XShell工具登录到开发服务器，找到部署到tomcat或者jar包目录，然后关闭服务，用Xftp工具打开这个目录，删除老的war包或者jar包，把刚刚打的新war包或jar包放到刚刚的目录，启动服务。

Jenkins可以自动帮我们完成上面的打包，上传，关闭服务，启动服务的操作，虽然可以定时设置一个时间点发布每天的代码，但是如果想要临时发布一个开发版本的话，就需要重新打开一个面板。

而我们的cloud toolkit插件可以在不切出idea界面的情况下，也能做到一键部署。

1. 在Alibaba Cloud View界面，添加host

   ![image-20200301134117559](https://wx3.sinaimg.cn/mw690/006pJIq7gy1gcedzq5ylrj31440u0n0a.jpg)

2. 添加好服务器之后，我们就可以在服务器上面写好需要执行的脚本restart-report.sh

   ```shell
   #这是一个重启war包的脚步
   #!/bin/bash
   #source /etc/profile
   ps -ef | grep apache-tomcat-report-manage | grep -v "grep" | awk '{print $2}' | xargs kill -9
   cd /home/xlzx_mysql/apache-tomcat-report-manage/ 
   rm -rf webapps/*
   mv /home/sh/dev/report.war webapps/
   sh bin/startup.sh
   ```

3. 在Alibaba Cloud View界面中选择Upload，在After Upload中添加执行该脚本的命令

   ```shell
   sh /home/sh/restart-report.sh 
   ```

4. 到此，idea自动部署就配置好了。

   

   

   