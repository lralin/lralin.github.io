---
title:Zookeeper集群搭建教程
---
#### 一、Zookeeper原理简介

##### 1. Zookeeper设计目的

- 最终一致性：client不论连接到那个Server，展示给它的都是同一个视图。  

- 可靠性：具有简单、健壮、良好的性能、如果消息m被到一台服务器接收，那么消息m将被所有服务器接收。
- 实时性：Zookeeper保证客户端将在一个时间间隔范围内获得服务器的更新信息，或者服务器失效的信息。但由于网络延时等原因，Zookeeper不能保证两个客户端能同时得到刚更新的数据，如果需要最新数据，应该在读数据之前调用sync()接口。  

- 等待无关（wait-free）：慢的或者失效的client不得干预快速的client的请求，使得每个client都能有效的等待。  

- 原子性：更新只能成功或者失败，没有中间状态。  

- 顺序性：包括全局有序和偏序两种：全局有序是指如果在一台服务器上消息a在消息b前发布，则在所有Server上消息a都将在消息b前被发布；偏序是指如果一个消息b在消息a后被同一个发送者发布，a必将排在b前面。

##### 2. Zookeeper工作原理

- 在zookeeper的集群中，各个节点共有下面3种角色和4种状态：   
角色：leader,follower,observer  
状态：leading,following,observing,looking

- Zookeeper的核心是原子广播，这个机制保证了各个Server之间的同步。实现这个机制的协议叫做Zab协议（ZooKeeper Atomic Broadcast protocol）。Zab协议有两种模式，它们分别是恢复模式（Recovery选主）和广播模式（Broadcast同步）。当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数Server完成了和leader的状态同步以后，恢复模式就结束了。状态同步保证了leader和Server具有相同的系统状态。

- 为了保证事务的顺序一致性，zookeeper采用了递增的事务id号（zxid）来标识事务。所有的提议（proposal）都在被提出的时候加上了zxid。实现中zxid是一个64位的数字，它高32位是epoch用来标识leader关系是否改变，每次一个leader被选出来，它都会有一个新的epoch，标识当前属于那个leader的统治时期。低32位用于递增计数。

- 每个Server在工作过程中有4种状态：  
***LOOKING***：当前Server不知道leader是谁，正在搜寻。  
***LEADING***：当前Server即为选举出来的leader。   
***FOLLOWING***：leader已经选举出来，当前Server与之同步。   
***OBSERVING***：observer的行为在大多数情况下与follower完全一致，但是他们不参加选举和投票，而仅仅接受(observing)选举和投票的结果。  
#### 二、Zookeeper安装
> Zookeeper链接：http://zookeeper.apache.org/
```
wget http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.8/zookeeper-3.4.8.tar.gz -P /usr/local/src/
tar zxvf zookeeper-3.4.8.tar.gz -C /opt
cd /opt && mv zookeeper-3.4.8 zookeeper
cd zookeeper
cp conf/zoo_sample.cfg conf/zoo.cfg

#把zookeeper加入到环境变量(为验证可以不加)
echo -e "# append zk_env\nexport PATH=$PATH:/opt/zookeeper/bin" >> /etc/profile
```
#### 三、Zookeeper集群配置
> ###### 等集群的所有机器启动之后再查看zookeeper的状态。
##### 1. Zookeeper配置文件修改
###### 修改过后的配置文件zoo.cfg，如下：
```
egrep -v "^#|^$" zoo.cfg
```
```
tickTime=2000
initLimit=10
syncLimit=5
dataLogDir=/opt/zookeeper/logs
dataDir=/opt/zookeeper/data
clientPort=2181
autopurge.snapRetainCount=500 #设置保留多少个snapshot
autopurge.purgeInterval=24 #设置多少小时清理一次
server.1= 192.168.1.148:2888:3888
server.2= 192.168.1.149:2888:3888
server.3= 192.168.1.150:2888:3888

```
###### 创建相关目录，三台节点都需要
```
mkdir -p /opt/zookeeper/{logs,data}
```
###### 其余zookeeper节点安装完成之后，同步配置文件zoo.cfg。

##### *如果要增加一台observer，上面的配置忽略，应按下面的配置。*
- ###### 不是observer的三台zoo.cfg的配置为：
```
tickTime=2000
initLimit=10
syncLimit=5
dataLogDir=/opt/zookeeper/logs
dataDir=/opt/zookeeper/data
clientPort=2181

autopurge.snapRetainCount=500 #设置保留多少个snapshot

autopurge.purgeInterval=24 #设置多少小时清理一次

server.1= 192.168.1.148:2888:3888
server.2= 192.168.1.149:2888:3888
server.3= 192.168.1.150:2888:3888
server.4= 192.168.1.151:2888:3888:observer
 #告诉集群192.168.1.151是observer模式

```
- ###### 192.168.1.151的observer配置为：
```
tickTime=2000
initLimit=10
syncLimit=5
dataLogDir=/opt/zookeeper/logs
dataDir=/opt/zookeeper/data
clientPort=2181
autopurge.snapRetainCount=500 #设置保留多少个snapshot
autopurge.purgeInterval=24 #设置多少小时清理一次

peerType=observer#观察者observer需要增加的配置 如果192.168.1.151这台机器是observer模式的话

server.1= 192.168.1.148:2888:3888
server.2= 192.168.1.149:2888:3888
server.3= 192.168.1.150:2888:3888
server.4= 192.168.1.151:2888:3888:observer
```
  
  
##### 2. 配置参数说明

- ***autopurge.snapRetainCount，autopurge.purgeInterval：***
客户端在与zookeeper交互过程中会产生非常多的日志，而且zookeeper也会将内存中的数据作为snapshot保存下来，这些数据是不会被自动删除的，这样磁盘中这样的数据就会越来越多。不过可以通过这两个参数来设置，让zookeeper自动删除数据。autopurge.purgeInterval就是设置多少小时清理一次。而autopurge.snapRetainCount是设置保留多少个snapshot，之前的则删除。
- ***tickTime***这个时间是作为zookeeper服务器之间或客户端与服务器之间维持心跳的时间间隔,也就是说每个tickTime时间就会发送一个心跳。
- ***initLimit***这个配置项是用来配置zookeeper接受客户端（这里所说的客户端不是用户连接zookeeper服务器的客户端,而是zookeeper服务器集群中连接到leader的follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。<br>当已经超过10个心跳的时间（也就是tickTime）长度后 zookeeper 服务器还没有收到客户端的返回信息,那么表明这个客户端连接失败。总的时间长度就是 10*2000=20秒。
- ***syncLimit***这个配置项标识leader与follower之间发送消息,请求和应答时间长度,最长不能超过多少个tickTime的时间长度,总的时间长度就是5*2000=10秒。
- ***dataDir***顾名思义就是zookeeper保存数据的目录,默认情况下zookeeper将写数据的日志文件也保存在这个目录里；
- ***clientPort***这个端口就是客户端连接Zookeeper服务器的端口,Zookeeper会监听这个端口接受客户端的访问请求；
- ***server.A=B:C:D***中的A是一个数字,表示这个是第几号服务器,B是这个服务器的IP地址，C第一个端口用来集群成员的信息交换,表示这个服务器与集群中的leader服务器交换信息的端口，D是在leader挂掉时专门用来进行选举leader所用的端口。
#####  3.3 创建ServerID标识

除了修改zoo.cfg配置文件外,zookeeper集群模式下还要配置一个myid文件,这个文件需要放在dataDir目录下。

这个文件里面有一个数据就是A的值（该A就是zoo.cfg文件中server.A=B:C:D中的A）,在zoo.cfg文件中配置的dataDir路径中创建myid文件。

#在192.168.1.148服务器上面创建myid文件，并设置值为1，同时与zoo.cfg文件里面的server.1保持一致，如下
```
echo "1" > /opt/zookeeper/data/myid
```
在192.168.1.149服务器上面创建myid文件，并设置值为2，同时与zoo.cfg文件里面的server.2保持一致，如下
```
echo "2" > /opt/zookeeper/data/myid
```
在192.168.1.150服务器上面创建myid文件，并设置值为3，同时与zoo.cfg文件里面的server.3保持一致，如下
```
echo "3" > /opt/zookeeper/data/myid
```

如果需要observer 在192.168.1.151服务器上面创建myid文件，并设置值为4，同时与zoo.cfg文件里面的server.4保持一致，如下
```
echo "4" > /opt/zookeeper/data/myid
```
到此，相关配置已完成  

#### 四、Zookeeper集群查看

#####  1.启动每个服务器上面的zookeeper节点：
```
/opt/zookeeper/bin/zkServer.sh start
```
#####  2、启动完成之后查看每个节点的状态：
```

[root@server1 bin]# /opt/zookeeper/bin/zkServer.sh status
JMX enabled by default
Using config: /home/xlzx/otter/zookeeper/bin/../conf/zoo.cfg
Mode: follower #显示follower表示从，显示leader表示主，显示observer表示观察者
```