---
title: Python3安装教程
date: 2020/01/05 12:15:25
updated: 2020/01/05 12:15:25
tags:
- python
- linux

---

##### 安装依赖包

1. 首先安装gcc编译器，gcc有些系统版本已经默认安装，通过  gcc --version  查看，没安装的先安装gcc。

   ```shell
   yum -y install gcc
   ```

   <!--more-->

2. 安装其它依赖包，（注：不要缺少，否则有可能安装python出错，python3.7.0以下的版本可不装 libffi-devel ）

   ```shell
   yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel
   ```

##### 下载python3.7.0源码，根据需求下载

1. 在`https://www.python.org/ftp/python/`中选择自己需要的python源码包，我下载的是python3.8.1

   ```shell
   wget https://www.python.org/ftp/python/3.8.1/Python-3.8.1.tgz
   ```

###### 解压Python-3.8.1.tgz

```shell
tar -zxvf Python-3.8.1.tgz
```

##### 建立一个空文件夹，用于存放python3程序

```shell
mkdir /usr/local/python3
```

##### 执行配置文件，编译，编译安装

```shell
cd Python-3.8.1
./configure --prefix=/usr/local/python3 --with-ssl
make && make install
```

安装完成没有提示错误便安装成功了，加with-ssl是为了保证有些源必须用https访问。

##### 建立软连接

```shell
ln -s /usr/local/python3/bin/python3.8 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3.8 /usr/bin/pip3
```

##### 测试一下python3是否可以用

```shell
python3 --version
```

##### 配置 国内源

```shell
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple [安装包名称]
```

可选源：

1. 阿里云`http://mirrors.aliyun.com/pypi/simple/`
2. 豆瓣`http://pypi.douban.com/simple/`
3. 清华大学`https://pypi.tuna.tsinghua.edu.cn/simple/`
4. 中国科学技术大学`http://pypi.mirrors.ustc.edu.cn/simple/`
5. 华中科技大学`http://pypi.hustunique.com/`

