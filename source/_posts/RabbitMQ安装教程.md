---
title: RabbitMQ安装教程
date: 2019/11/30 12:50:25
updated: 2019/11/30 12:50:25
tags:
- RabbitMQ
- 消息队列
---

##### 脚本安装

1. 安装[Erlang](https://packagecloud.io/rabbitmq/erlang/install#bash-rpm)

   - 执行配置脚本

     ```shell
     curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash
     ```

   - 运行yum安装

     ```shell
     yum install erlang
     ```

   <!--more-->

2. 安装[RabbitMQ](https://packagecloud.io/rabbitmq/rabbitmq-server/install#bash-rpm)

   - 执行配置脚本

     ```shell
     curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
     ```

   - 运行yum安装

     ```shell
     yum install rabbitmq-server
     ```

3. 安装完成，运行服务

   ```shell
   #开机启动
   chkconfig rabbitmq-server on
   
   #启动服务
   /sbin/service rabbitmq-server start
   #停止服务
   /sbin/service rabbitmq-server stop
   ```

##### rpm离线安装

1. 下载[Erlang](https://bintray.com/rabbitmq-erlang/rpm/download_file?file_path=erlang%2F22%2Fel%2F7%2Fx86_64%2Ferlang-22.1.8-1.el7.x86_64.rpm)安装包

   ```shell
   rpm -ivh erlang-22.1.8-1.el7.x86_64.rpm
   ```

2. 下载[rabbitmq](https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.1/rabbitmq-server-3.8.1-1.el7.noarch.rpm)安装包

   ```shell
   rpm -ivh rabbitmq-server-3.8.1-1.el7.noarch.rpm
   ```

   

##### 集群构建指南

[官方安装教程](https://www.rabbitmq.com/clustering.html)

1. RabbitMQ节点使用短域名或完全合格（FQDN）域名相互寻址。因此，所有群集成员的主机名以及所有可能使用命令行工具（例如rabbitmqctl）的机器都必须是可解析的。

   主机名解析可以使用操作系统提供的任何标准方法：
   
      * DNS记录 
   
      * 本地主机文件（例如/etc/hosts）。配置为：
   
        ```
        192.168.3.1 rabbit1
        ```

2. RabbitMQ节点和CLI工具（例如rabbitmqctl）使用cookie来确定是否允许它们彼此通信。为了使两个节点能够通信，它们必须具有称为Erlang cookie的相同共享机密。

    Cookie只是一串字母数字字符，最大长度为255个字符。它通常存储在本地文件中。所有者必须只能访问该文件（例如，具有UNIX权限600或类似权限）。所以想要进行集群之间的通信，每个群集节点必须具有相同的cookie。
    
    Linux, MacOS, *BSD系统的cookie位置一般放在`/var/lib/rabbitmq/.erlang.cookie`或者`$HOME/.erlang.cookie`
    
    Windows系统，Erlang 20.2之后的版本一般放在`C:\Windows\system32\config\systemprofile\.erlang.cookie`或者`C:\Users\%USERNAME%\.erlang.cookie`;而Erlang 19.3 到 20.2之间的会在`C:\Windows\.erlang.cookie`或者`C:\Users\%USERNAME%\.erlang.cookie`

3. 配置好hosts和cookie之后，我们正式开始搭建集群。假如我们现在又三台服务器分别为rabbit1、rabbit2、rabbit3。

   - 首先我们使用 `rabbitmq-server -detached`启动rabbit1机器。

   - 然后把rabbit2机器加入到集群`rabbitmqctl join-cluster rabbit@rabbit1`,并启动`rabbitmqctl start_app`

   - 用相同的方式把rabbit3放入集群中，然后查看集群状态：

     ```shell
     # on rabbit1
     rabbitmqctl cluster_status
     # => Cluster status of node rabbit@rabbit1 ...
     # => [{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
     # =>  {running_nodes,[rabbit@rabbit3,rabbit@rabbit2,rabbit@rabbit1]}]
     # => ...done.
     
     # on rabbit2
     rabbitmqctl cluster_status
     # => Cluster status of node rabbit@rabbit2 ...
     # => [{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
     # =>  {running_nodes,[rabbit@rabbit3,rabbit@rabbit1,rabbit@rabbit2]}]
     # => ...done.
     
     # on rabbit3
     rabbitmqctl cluster_status
     # => Cluster status of node rabbit@rabbit3 ...
     # => [{nodes,[{disc,[rabbit@rabbit3,rabbit@rabbit2,rabbit@rabbit1]}]},
     # =>  {running_nodes,[rabbit@rabbit2,rabbit@rabbit1,rabbit@rabbit3]}]
     # => ...done.
     ```

   - 可以看到我们每个节点都是disc磁盘类型。

     节点可以是磁盘节点或RAM节点。 （注意：磁盘和RAM可以互换使用）。 RAM节点仅将数据存储在内存中。这不包括消息，消息存储索引，队列索引和其他节点状态。
     在大多数情况下，您希望所有节点都是磁盘节点。 RAM节点是一种特殊情况，可用于提高队列，交换或绑定流失率较高的群集的性能。 RAM节点不提供更高的消息速率。如有疑问，请仅使用磁盘节点。
     由于RAM节点仅将内部数据库表存储在RAM中，因此它们必须在启动时从对等节点同步它们。这意味着一个群集必须至少包含一个磁盘节点。因此，不可能手动删除群集中最后剩余的磁盘节点。

   - 到此我们集群就创建完成了。