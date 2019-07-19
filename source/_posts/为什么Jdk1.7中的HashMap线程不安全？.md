---
title: 为什么Jdk1.7中的HashMap线程不安全？
date: 2019-07-07 22:05:25
updated: 2019-07-07 22:05:25
tags: 
- HashMap
- java
---
### jdk1.7 HashMap线程不安全
- rehash的时候形成环形链表，导致用get方法获取一个不存在的key时，会发生死循环。
- 造成数据丢失。

**1. 前提**

- hashMap由数组和链表组成
- hashMap在到达一定阈值的时候会进行扩容
- 扩容的时候会把当前桶的记录重新计算hash放入新桶中
<!--more-->
- 重制大小的时候，数据转移的transfer方法
```java 
void transfer(Entry[] newTable) {
     Entry[] src = table;                   //src引用了旧的Entry数组
     int newCapacity = newTable.length;
     for (int j = 0; j < src.length; j++) { //遍历旧的Entry数组
        Entry<K,V> e = src[j];             //取得旧Entry数组的每个元素
         if (e != null) {
             src[j] = null;//释放旧Entry数组的对象引用（for循环后，旧的Entry数组不再引用任何对象）
             do {
                Entry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity); //！！重新计算每个元素在数组中的位置
                e.next = newTable[i]; //标记[1]
                newTable[i] = e;      //将元素放在数组上
                e = next;             //访问下一个Entry链上的元素
             } while (e != null);
         }
     }
 } 
 //newTable[i]的引用赋给了e.next，也就是使用了单链表的头插入方式，同一位置上新元素总会被放在链表的头部位置；
 这样先放在一个索引上的元素终会被放到Entry链的尾部。（PS:在key不同但是桶位置相同的情况下，就会把数据放在这个链表上）
```

**2. 场景**

```java
public class HashMapInfiniteLoop {  

    private static HashMap<Integer,String> map = new HashMap<Integer,String>(2，0.75f);  
    public static void main(String[] args) {  
        map.put(5， "C");  

        new Thread("Thread1") {  
            public void run() {  
                map.put(7, "B");  
                System.out.println(map);  
            };  
        }.start();  
        new Thread("Thread2") {  
            public void run() {  
                map.put(3, "A);  
                System.out.println(map);  
            };  
        }.start();        
    }  
} 
```
其中，map初始化为一个长度为2的数组，loadFactor=0.75，threshold=2*0.75=1，也就是说当put第二个key的时候，map就需要进行resize。

通过设置断点让线程1和线程2同时debug到transfer方法的首行。注意此时两个线程已经成功添加数据。放开thread1的断点至transfer方法的“Entry next = e.next;” 这一行；然后放开线程2的的断点，让线程2进行resize。结果如下图。

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/7df99266.png)

注意，Thread1的 e 指向了key(3)，而next指向了key(7)，其在线程二rehash后，指向了线程二重组后的链表。

线程一被调度回来执行，执行 newTalbe[i] = e， 

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/4c3c28fb.png)

然后是e = next，导致了e指向了key(7)，而下一次循环的next = e.next导致了next指向了key(3)。

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/6c8d086a.png)

e.next = newTable[i] 导致 key(3).next 指向了 key(7)。注意：此时的key(7).next 已经指向了key(3)， 环形链表就这样出现了，而再往下就是null了，从而key(5)就被丢失了。

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/6eed9aaf.png)

于是，当我们用线程一调用map.get(11)时，悲剧就出现了——Infinite Loop。
如果不是很明白上面的逻辑的话，请多看几遍，而且需要先把前提理论理解清楚。

**3. 参考链接**

看了很多篇文章，[这篇](https://tech.meituan.com/2016/06/24/java-hashmap.html)突然让我顿悟。
