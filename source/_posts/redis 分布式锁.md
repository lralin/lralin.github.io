---
title: redis 分布式锁
date: 2019-07-18 17:57:25
updated: 2019-07-19 15:03:25
tags: 
- java
- redis
---

### 基于redis 集群或单击的分布式锁

随着分布式服务的广泛应用，分布式锁也变成了一项必修课，下面我们实现了一个redis分布式锁。
<!-- more -->
1. 获取锁(非阻塞)
```java
    public boolean tryGetDistributedLock(String lockKey, String requestId, long expireTime) throws Exception {
            String result = jedisCluster.set(lockKey, requestId, "NX", "PX", expireTime);
            
            if ("OK".equals(result)) {
                return true;
            }
            return false;
    }
```

获取锁(阻塞)
```java
    /**
     * blocking lock(阻塞锁)
     *
     * @param lockKey
     * @param requestId
     * @param expireTime
     */
    public void tryBlockLock(String lockKey, String requestId, long expireTime) throws Exception 	 {
        for (; ;) {
            String result = jedisCluster.set(lockKey, requestId, "NX", "PX", expireTime);
            if ("OK".equals(result)) {
                 break;
            }
        }
    }
```
利用 Redis set key 时的一个 NX 参数可以保证在这个 key 不存在的情况下写入成功。并且再加上 EX 参数可以让该 key 在超时之后自动删除。

所以利用以上两个特性可以保证在同一时刻只会有一个进程获得锁，并且不会出现死锁(最坏的情况就是超时自动删除 key)。

2. 释放锁
```java
    public boolean releaseDistributedLock(String lockKey, String requestId) throws Exception {
            String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
            Object result = jedisCluster.eval(script, Collections.singletonList(lockKey),
                    Collections.singletonList(requestId));
            if ("1".equals(result)) {
                return true;
            }
            return false;
    }
```

因为分布式服务，机器A获取的锁，不应该被机器B解锁，所以最好的方式是在每次解锁时都需要判断锁是否是自己的。这时就需要结合加锁机制一起实现了。加锁时需要传递一个参数，将该参数作为这个 key 的 value，这样每次解锁时判断 value 是否相等即可。

3. 订阅sub
```java
     public String subscribe(Map<String, Object> callBackMap, String channels) throws Exception{
            if (callBackMap == null) {
                callBackMap = new HashMap<String, Object>();
            }
            jedisCluster.subscribe(new YmRedisMQHandler(callBackMap), channels);
            Object obj = callBackMap.get("status");
            return obj==null?"":String.valueOf(obj);
    }
```

YmRedisMQHandler类

```java
    import java.io.Serializable;
    import java.util.Map;
    
    import redis.clients.jedis.JedisPubSub;
    
    /**
     * 消息监听器类
     * 
     * @version 2017年11月1日上午8:50:29
     * @author linrui.li
     */
    public class YmRedisMQHandler extends JedisPubSub implements Serializable{
        private static final long serialVersionUID = 3981666970819669711L;
        private Map<String, Object> param;
        
        public YmRedisMQHandler(Map<String, Object> param) {
            super();
            this.param = param;
        }
        
        @Override
        public void onMessage(String channel, String message) {
            param.put("status", message);
            this.unsubscribe();
        }
    }
```

4. 通知pub
```java
    public Long publish(String channel, String message) throws Exception {
            return jedisCluster.publish(channel, message);
    }
```

### 分布式服务 进行api请求获取token

1. 非阻塞锁，加上订阅模式，设计token获取，防止高并发的时候多次进行http请求获取token。
缺点:如果锁里面的http请求获取token失败了，会导致当前订阅的线程全部失败。
```java
    private String getTokenHandler(RouteBean routeBean) throws Exception {
        String pubKey = "";
        Map<String, Object> callBackMap = new HashMap<String, Object>();
        int timeout = Integer.parseInt("10") * 1000;
        //1. 直接从redis获取
        pubKey = cacheDubboService.get(Constant.TOKEN_KEY);
        if (StringUtils.isNotBlank(pubKey)) {
            return pubKey;
        }
        String requestId = "locks_" + getHostName() + System.currentTimeMillis();
        //2. 获取锁，true则得到锁，否则订阅阻塞
        if (cacheDubboService.tryGetDistributedLock(Constant.LOCK_KEY, requestId, timeout + 2000)) {
            String status = "fail";
            try {
                //3. 防止在第一步调用后，在第二步得到锁后，上一次http获取token已经成功了。所以进行再一次的get
                pubKey = cacheDubboService.get(Constant.TOKEN_KEY);
                if (StringUtils.isNotBlank(pubKey)) {
                    return pubKey;
                }
                //4. 获取token
                log.info("http获取token");
                //模拟获取到token
                Thread.sleep(200);
                pubKey = "getTokenId(routeBean)" + System.currentTimeMillis();

                if (StringUtils.isNotBlank(pubKey)) {
                    status = "success";
                    //5. 设置Token到缓存
                    // 未实现：Token 该tokenId的失效时间为最后一次数据接口访问往后推24小时，如果之间一直有接口调用，那么该tokenId将一直有效
                    Integer time = 7200;
                    cacheDubboService.setex(Constant.TOKEN_KEY, pubKey, time);
                } else {
                    status = "fail";
                    log.info("获取token失败");
                }
            } catch (Exception e) {
                log.error("获取token异常", e);
            } finally {
                //6. 通知订阅者获取Token
                cacheDubboService.publish(Constant.PUB_SUB, status);
                //7. 释放锁：因为是非阻塞式锁，所以只有对获取到锁的进行释放就好。
                cacheDubboService.releaseDistributedLock(Constant.LOCK_KEY, requestId);
            }
        } else {
            log.info("订阅消息");
            //8.  订阅消息 阻塞，等待通知。
            String subscribe = cacheDubboService.subscribe(callBackMap, Constant.PUB_SUB);// 会阻塞
            log.info("得到的通知结果，subscribe:{}", subscribe);
            if (!"success".equals(subscribe)) {
                log.error("token获取失败。");
                return "";
            }
            pubKey = cacheDubboService.get(Constant.TOKEN_KEY);
        }

        return pubKey;
    }
    
    /**
     * 获取计算机名称
     *
     * @param routeBean
     * @return
     * @version 2017年12月11日下午4:47:16
     */
    public static String getHostName() {
        if (System.getenv("COMPUTERNAME") != null) {
            return System.getenv("COMPUTERNAME");
        }
        else {
            return getHostNameForLiunx();
        }
    }
    
    /**
     * 获取linux 主机名
     *
     * @param routeBean
     * @return
     * @version 2017年12月11日下午4:47:16
     */
    public static String getHostNameForLiunx() {
        try {
            //java.net.InetAddress;
            return (InetAddress.getLocalHost()).getHostName();
        }
        catch (UnknownHostException uhe) {
            String host = uhe.getMessage(); // host = "hostname: hostname"
            if (host != null) {
                int colon = host.indexOf(':');
                if (colon > 0) {
                    return host.substring(0, colon);
                }
            }
            return "hostname";
        }
    }
```

```java
    public class Constant {
        /*redis存token的key*/
        public static final String TOKEN_KEY = "test:redis:tokenkey";
        public static final String LOCK_KEY = "test:redis:lockkey";
        public static final String PUB_SUB = "test:redis:pubsubkey";
    }
```

2. 通过阻塞式锁，获取token。
```java
	/**
     * 防止高并发获取token
     *
     * @param routeBean
     * @return
     * @version 2019年07月19日下午4:47:16
     */
    private String getTokenLock(RouteBean routeBean) throws Throwable {
        String pubKey = "";
        //1. 从缓存中获取token
        pubKey = cacheDubboService.get(Constant.TOKEN_KEY);
        if (StringUtils.isNotBlank(pubKey)) {
            log.info("第一次通过cacheDubboService获取token");
            return pubKey;
        }
        String requestId = "locks_" + getHostName() + System.currentTimeMillis();
        try {
            //2. 缓存中获取token为空，则进行http请求，获取最新token
            //读取配置文件中的超时时间
            int timeout = Integer.parseInt("10") * 1000;
            // 3. 加锁，从而只有一个线程去获取最新的token。
            cacheDubboService.tryBlockLock(Constant.LOCK_KEY, requestId,timeout + 2000)
            
            pubKey = cacheDubboService.get(Constant.TOKEN_KEY);
            if (StringUtils.isNotBlank(pubKey)) {
                log.info("第二次通过cacheDubboService获取token");
                return pubKey;
            }

            //通过http获取token
            log.info("通过http获取token");
            Thread.sleep(2000);
            pubKey = "getTokenId(routeBean)" + System.currentTimeMillis();

            if (StringUtils.isNotBlank(pubKey)) {
                // 设置Token到缓存 Token 该tokenId的失效时间为最后一次数据接口访问往后推24小时，如果之间一直有接口调用，那么该tokenId将一直有效
                Integer time = 7200;
                cacheDubboService.setex(Constant.TOKEN_KEY, pubKey, time);
                return pubKey;
            } else {
                log.info("获取token失败");
                cacheDubboService.del(Constant.TOKEN_KEY);
            }

        } catch (Throwable e) {
            log.error("http获取token异常", e);
        } finally {
            // 释放锁
            cacheDubboService.releaseDistributedLock(Constant.LOCK_KEY, requestId);
        }

        return pubKey;
    }
```

### 优点

- 使用 Redis 可以保证性能。
- 阻塞锁与非阻塞锁见上文。
- 利用超时机制解决了死锁。
- Redis 支持集群部署提高了可用性。

### 参考

[crossoverJie](https://crossoverjie.top/2018/03/29/distributed-lock/distributed-lock-redis/)

[baidu-dlock](https://github.com/baidu/dlock/blob/master/README.zh_cn.md)