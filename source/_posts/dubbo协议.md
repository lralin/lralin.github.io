---
title: dubbo协议
date: 2020-03-15 18:35:25
updated: 2020-03-15 18:35:25
tags: 
- dubbo
- hessian
---

### dubbo协议

dubbo的默认协议，单一长连接和NIO异步通讯，适合少数据高并发，消费者大于提供者的应用场景。使用单连接的原因是防止消费者太多直接压垮服务者，而长连接是为了减少握手验证的次数，并使用异步IO，复用线程池。

<!--more-->

### rmi协议

rmi协议是阻塞式短连接和JDK标准序列化，采用多连接，短连接，TCP同步传输，适合传入传出参数数据包大小混合，消费者与提供者个数差不多，可传文件。

### hessian协议

hessian底层采用http通讯，采用多连接，短连接，同步传输，适合传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件。

### http协议

采用Spring的HttpInvoker实现，也是多连接，短连接，同步传输，传入传出参数混合大小，提供者比消费者个数多，可浏览器查看，不支持传文件。

### webservice协议

基于Apache CXF，多连接，短连接，同步传输，适用于系统集成，跨语言调用。

### thrift协议

thrift 协议是对 thrift 原生协议的扩展，[Thrift](http://thrift.apache.org/) 是 Facebook 捐给 Apache 的一个 RPC 框架，不能在协议中传递 null 值。

###memcached协议

基于 memcached 实现的 RPC 协议，[Memcached](http://memcached.org/) 是一个高效的 KV 缓存服务器。

### redis协议

基于 Redis 实现的 RPC 协议，[Redis](http://redis.io/) 是一个高效的 KV 存储服务器。

###rest协议

基于标准的Java REST API，实现过程类似Spring MVC