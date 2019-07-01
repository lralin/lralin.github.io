---
title: Jenkins自动打包
date: 2019-06-21 15:55:25
updated: 2019-06-21 15:55:25
tags: 
- Jenkins
- tomcat
---

### 配置远程服务器

点击 **Manage Jenkins** ，选择  **Configure System** 服务器配置
![Configure](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/jenkins/Jenkins-1.png)

<!--more-->

找到 **Publish over SSH** 配置远程服务器
![SSH](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/jenkins/Jenkins-2.png)
其中 Remote Directory 表示连上服务器后，默认进入的目录。文件传输的时候，目标目录会加上这个前缀部分，所以这个目录配置的不要太深，我们这里配置为`/home/jenkins/build_code/`。
登录有密码登录和免密登录，二选其一就好。

### 配置环境插件（jdk和maven）

点击 **Manage Jenkins** ，选择 **Global Tool Configuration** ，

![Global Tool](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/jenkins/Jenkins-3.png)

进行jdk配置，

![Maven](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/jenkins/Jenkins-4.png)

配置Maven

![Maven](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/jenkins/Jenkins-5.png)

### 配置项目自动打包

1. 首页点击 **New Item**  

   ![Maven](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/jenkins/Jenkins-12.png)

   配置项目名并选择 **maven project** ，如果之前有过相似的项目，也可以配置 **Copy from**

   ![Maven](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/jenkins/Jenkins-13.png)

2. 配置 **General** ，Description可以记录一下项目内容，方便以后查看。jdk选择之前配置的。

   ![Maven](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/jenkins/Jenkins-6.png)

3. **Source Code Management** 如果你代码在gitlab或者github就选择git，我这里选择Subversion，复制项目的svn路径到Repository URL，用svn的账号密码配置好Credentials，其他的默认就好。

   ![Maven](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/jenkins/Jenkins-7.png)

4. **Build Triggers** 选择Poll SCM 调度任务 `H/5 * * * *` 设置为五分钟检测一次项目路径下svn是否有变化，有变化的话就执行打包任务。**Build Environment** 勾上Add timestamps to the Console Output，这样打包的时候执行的命令行任务信息会在console里面打印出来，有利于查看执行过程

   ![Maven](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/jenkins/Jenkins-14.png)

5. **Pre Steps** 打包之前执行的命令，可以不设置，也可以设置关闭该项目服务的命令，我们这里不设置，统一在 **Post Steps** 里面处理。

6. **Build** maven命令打包

    - Root POM 默认配置pom.xml就好。
    - Goals and options 可以增加跳过测试 `-Dmaven.test.skip=true` ，强制从私服拿最新的jar包 `-U` 等maven命令。`clean install -Dmaven.test.skip=true -U`

    ![Maven](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/jenkins/Jenkins-15.png)

7. **Post Steps** 打包之后执行的命令，我们之前配置了远程服务，这里就可以配置远程服务执行的脚本了。主要的配置在*Transfers*里面。

   - *Source files*需要填的是该项目打完包之后的war包位置，一般是直接在target下面，而我这个项目是父子结构的项目，war包是在一个子项目的target下面。
   - *Remove prefix*理所当然就是需要去掉的前缀，这样我们把war发送到远程服务器的时候就不会增加更深的目录了。
   - *Remote directory* 远程目录，不填就会放在ssh服务器配置时候的目录，具体可以通过 **Configure System** 进行查看；填了的话也是在当前目录前面再加上远程服务器配置的目录，例如我们这里填`tomcat/manage`,那么我们的war包会被传送到`/home/jenkins/build_code/tomcat/manage` 目录下面，我们这里选择不填。
   - *Exec command* 需要执行的命令，我们这里设置执行一个脚本，然后我们在脚本中写具体的操作步骤，大概的内容就是把首先关闭服务，备份并移除tomcat下老的war包，把远程打包过来的war包移动到tomcat的webapp下面，启动项目，五秒之后打印日志。
   - 脚本manage.sh
   ```shell
   #!/bin/bash
   # jdk配置，当直接使用java命令进行项目启动的时候可以设置
   #export JAVA_HOME PATH CLASSPATH
   #JAVA_HOME=/usr/lib/jvm/jdk1.7.0_79
   #PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
   #CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH

   #war名称
   ServiceName=manage
   #构建目录
   build_code_dir=/home/jenkins/build_code/
   # tomcat目录
   tomcat_dir=/home/jenkins/tomcat/tomcat-manage
   cd $tomcat_dir
   # stop tomcat
   ./bin/shutdown.sh
   sleep 2
   # 使用ps命令保证项目已经被停止
   pidlist=`ps -ef|grep $tomcat_dir|grep -v "grep"|awk '{print $2}'`
   function stop(){
   if [ "$pidlist" == "" ]
     then
       echo "---- $tomcat_dir 已经关闭----"
    else
       echo "-- $tomcat_dir 进程号 :$pidlist"
       kill -9 $pidlist
       echo "--KILL $pidlist:"
   fi
   }
   # 执行stop方法
   stop
   sleep 1

   # 移除tomcat老的项目文件，这里也可以先备份，由于是开发环境所以我们就不进行备份了。
   rm -rf ${tomcat_dir}/webapps/${ServiceName}*
   # 从发布的目录把对应项目移动当前目录(DIR)
   cp -rf ${build_code_dir}/${ServiceName}.war ${tomcat_dir}/webapps

   rm -rf ${build_code_dir}/${ServiceName}.war

   # 启动新项目
   ./bin/startup.sh
   ```

8. 自动部署成功。