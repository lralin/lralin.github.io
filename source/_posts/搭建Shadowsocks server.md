---
title: 搭建Shadowsocks server  
date: 2018-02-06 10:09:25
---
#### 一、安装 shadowsocks
- Debian / Ubuntu 用户运行：  
```
 apt-get install python-pip
 pip install shadowsocks
```
- CentOS 用户运行： 
```
yum install python-setuptools && easy_install pip
pip install shadowsocks
```
<!--more-->
#### 二、配置文件  
- 新建配置文件：  
```
vim ~/shadowsocks/config.json
```
- 编辑配置文件，依然是按 i 进入编辑，按 ESC 退出编辑，按 :wq 保存并退出：
```
{
"server":"0.0.0.0",
"local_address":"127.0.0.1",
"local_port":1080,
"port_password":{
"8392":"youpassword",
"8393":"youpassword",
"8394":"youpassword",
"8395":"youpassword",
"8396":"youpassword",
"8397":"youpassword" # 多端口设置
},
"timeout":300,
"method":"aes-256-cfb",
"fast_open": false
}
```
#### 三、启动并永久运行  
- 启动命令
```
nohup ssserver -c ~/shadowsocks/config.json -d start &
#nohup ssserver -c ~/shadowsocks/config.json > log -d start &
```
>  ssserver 是 shadowsocks 的服务端命令。
   -c 表示以配置文件的方式运行 shadowsocks， ~/shadowsocks/config.json 则是配置文件的路径。
   -d 表示在后台运行，这样允许用户进行其他操作。
   start 就是启动。
   nohup 以及最后的 & 是让 shadowsocks 服务端一直运行，并把运行日志输出到当前用户主目录下的 nohup.out 文件中。  
 
- 停止命令：
```
ssserver -c ~/shadowsocks/config.json -d stop
```
