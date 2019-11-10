---
title: dubbo负载均衡-ConsistentHashLoadBalance
date: 2019-11-02 09:43:25
updated: 2019-11-02 10:57:25
tags: 
- 负载均衡
- dubbo
---

### 源码分析

这是一个基于hash算法的负载均衡，首先根据每个节点的ip或者其他信息为节点生成一个hash，并将这个hash投射到[0, 2<sup>32</sup> - 1] 的圆环上。当有请求过来时，为该请求生成一个hash值。然后查找第一个大于或者等于该hash的缓存节点，并使用该节点的服务。如果该节点挂了，则在下一次查询或写入缓存时，为缓存项查找另一个大于其 hash 值的缓存节点即可。大致效果如下图所示，每个缓存节点在圆环上占据一个位置。如果缓存项的 key 的 hash 值小于缓存节点 hash 值，则到该缓存节点中存储或读取缓存项。比如下面绿色点对应的缓存项将会被存储到 cache-2 节点中。由于 cache-3 挂了，原本应该存到该节点中的缓存项最终会存储到 cache-4 节点中。

![图一](http://dubbo.apache.org/docs/zh-cn/source_code_guide/sources/images/consistent-hash.jpg)

下面来看看一致性 hash 在 Dubbo 中的应用。我们把上图的缓存节点替换成 Dubbo 的服务提供者，于是得到了下图：

![图二](http://dubbo.apache.org/docs/zh-cn/source_code_guide/sources/images/consistent-hash-invoker.jpg)

这里相同颜色的节点均属于同一个服务提供者，比如 Invoker1-1，Invoker1-2，……, Invoker1-160。这样做的目的是通过引入虚拟节点，让 Invoker 在圆环上分散开来，避免数据倾斜问题。所谓数据倾斜是指，由于节点不够分散，导致大量请求落到了同一个节点上，而其他节点只会接收到了少量请求的情况。比如：

![图三](http://dubbo.apache.org/docs/zh-cn/source_code_guide/sources/images/consistent-hash-data-incline.jpg)

如上，由于 Invoker-1 和 Invoker-2 在圆环上分布不均，导致系统中75%的请求都会落到 Invoker-1 上，只有 25% 的请求会落到 Invoker-2 上。解决这个问题办法是引入虚拟节点，通过虚拟节点均衡各个节点的请求量。

接下来开始分析源码。我们先从 ConsistentHashLoadBalance 的 doSelect 方法开始看起，如下：

```java
public class ConsistentHashLoadBalance extends AbstractLoadBalance {

    private final ConcurrentMap<String, ConsistentHashSelector<?>> selectors = 
        new ConcurrentHashMap<String, ConsistentHashSelector<?>>();

    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        String methodName = RpcUtils.getMethodName(invocation);
        String key = invokers.get(0).getUrl().getServiceKey() + "." + methodName;

        // 获取 invokers 原始的 hashcode
        int identityHashCode = System.identityHashCode(invokers);
        ConsistentHashSelector<T> selector = (ConsistentHashSelector<T>) selectors.get(key);
        // 如果 invokers 是一个新的 List 对象，意味着服务提供者数量发生了变化，可能新增也可能减少了。
        // 此时 selector.identityHashCode != identityHashCode 条件成立
        if (selector == null || selector.identityHashCode != identityHashCode) {
            // 创建新的 ConsistentHashSelector
            selectors.put(key, new ConsistentHashSelector<T>(invokers, methodName, identityHashCode));
            selector = (ConsistentHashSelector<T>) selectors.get(key);
        }

        // 调用 ConsistentHashSelector 的 select 方法选择 Invoker
        return selector.select(invocation);
    }
    
    private static final class ConsistentHashSelector<T> {...}
}
```

如上，doSelect 方法主要做了一些前置工作，比如检测 invokers 列表是不是变动过，以及创建 ConsistentHashSelector。这些工作做完后，接下来开始调用 ConsistentHashSelector 的 select 方法执行负载均衡逻辑。在分析 select 方法之前，我们先来看一下一致性 hash 选择器 ConsistentHashSelector 的初始化过程，如下：

```java
private static final class ConsistentHashSelector<T> {

    // 使用 TreeMap 存储 Invoker 虚拟节点
    private final TreeMap<Long, Invoker<T>> virtualInvokers;

    private final int replicaNumber;

    private final int identityHashCode;

    private final int[] argumentIndex;

    ConsistentHashSelector(List<Invoker<T>> invokers, String methodName, int identityHashCode) {
        this.virtualInvokers = new TreeMap<Long, Invoker<T>>();
        this.identityHashCode = identityHashCode;
        URL url = invokers.get(0).getUrl();
        // 获取虚拟节点数，默认为160
        this.replicaNumber = url.getMethodParameter(methodName, "hash.nodes", 160);
        // 获取参与 hash 计算的参数下标值，默认对第一个参数进行 hash 运算
        String[] index = Constants.COMMA_SPLIT_PATTERN.split(url.getMethodParameter(methodName, "hash.arguments", "0"));
        argumentIndex = new int[index.length];
        for (int i = 0; i < index.length; i++) {
            argumentIndex[i] = Integer.parseInt(index[i]);
        }
        for (Invoker<T> invoker : invokers) {
            String address = invoker.getUrl().getAddress();
            for (int i = 0; i < replicaNumber / 4; i++) {
                // 对 address + i 进行 md5 运算，得到一个长度为16的字节数组
                byte[] digest = md5(address + i);
                // 对 digest 部分字节进行4次 hash 运算，得到四个不同的 long 型正整数
                for (int h = 0; h < 4; h++) {
                    // h = 0 时，取 digest 中下标为 0 ~ 3 的4个字节进行位运算
                    // h = 1 时，取 digest 中下标为 4 ~ 7 的4个字节进行位运算
                    // h = 2, h = 3 时过程同上
                    long m = hash(digest, h);
                    // 将 hash 到 invoker 的映射关系存储到 virtualInvokers 中，
                    // virtualInvokers 需要提供高效的查询操作，因此选用 TreeMap 作为存储结构
                    virtualInvokers.put(m, invoker);
                }
            }
        }
    }
}
```

ConsistentHashSelector 的构造方法执行了一系列的初始化逻辑，比如从配置中获取虚拟节点数以及参与 hash 计算的参数下标，默认情况下只使用第一个参数进行 hash。需要特别说明的是，ConsistentHashLoadBalance 的负载均衡逻辑只受参数值影响，具有相同参数值的请求将会被分配给同一个服务提供者。ConsistentHashLoadBalance 不 关系权重，因此使用时需要注意一下。

在获取虚拟节点数和参数下标配置后，接下来要做的事情是计算虚拟节点 hash 值，并将虚拟节点存储到 TreeMap 中。到此，ConsistentHashSelector 初始化工作就完成了。接下来，我们来看看 select 方法的逻辑。

```java
public Invoker<T> select(Invocation invocation) {
    // 将参数转为 key
    String key = toKey(invocation.getArguments());
    // 对参数 key 进行 md5 运算
    byte[] digest = md5(key);
    // 取 digest 数组的前四个字节进行 hash 运算，再将 hash 值传给 selectForKey 方法，
    // 寻找合适的 Invoker
    return selectForKey(hash(digest, 0));
}

private Invoker<T> selectForKey(long hash) {
    // 到 TreeMap 中查找第一个节点值大于或等于当前 hash 的 Invoker
    Map.Entry<Long, Invoker<T>> entry = virtualInvokers.tailMap(hash, true).firstEntry();
    // 如果 hash 大于 Invoker 在圆环上最大的位置，此时 entry = null，
    // 需要将 TreeMap 的头节点赋值给 entry
    if (entry == null) {
        entry = virtualInvokers.firstEntry();
    }

    // 返回 Invoker
    return entry.getValue();
}
```

如上，首先是对参数进行 md5 以及 hash 运算，得到一个 hash 值。然后再拿这个值到 TreeMap 中查找目标 Invoker 即可。

到此关于 ConsistentHashLoadBalance 就分析完了。