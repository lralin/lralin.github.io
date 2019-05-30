---
title: Junit压力测试
date: 2019/05/30 12:15:25
updated: 2019/05/30 12:15:25
tags:
- junit
- 测试
- 性能测试
---

##### 简介

contiperf是一个性能测试工具，可以设置线程数`threads`，总笔数`invocations`，总时长`duration` 等参数，配合`@PerfTest` 注解自由设置，并且还会生成一个折线图，目录位置一般就在`target/contiperf-report` 目录下面。

<!--more-->
##### 示例

1. pom包：
```xml
   <dependency>
       <groupId>org.databene</groupId>
       <artifactId>contiperf</artifactId>
       <version>2.1.0</version>
       <scope>test</scope>
   </dependency>
```
2. Java代码中的实际调用关系
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:spring-dubbo-test.xml" })
public class PhoneverifyDubboTest {
	//使用@Rule注释激活ContiPerf
	@Rule
	public ContiPerfRule i = new ContiPerfRule();
	//加上junit的注解，表示该测试启动100个线程，一共跑5000次
	@Test
	@PerfTest(threads = 100, invocations = 5000)
	public void testTelecom() throws Throwable {
		System.out.println("lralin");
	}
	//表示该测试启动100个线程，一共执行50s
	@Test
	@PerfTest(threads = 100, duration = 50000)
	public void testTelecom() throws Throwable {
		System.out.println("lralin");
	}
}
```
3. 在`@PerfTest`配置中如果duration和invocations同时使用的话会在保证两个参数都符合的情况，才会停止。例如`@PerfTest(threads = 100, duration = 50000, invocations = 500)` ，如果50s内，500次请求还没结束的话，会一直等到500次执行完毕才会结束.
4. 关于timer参数，主要是用于设置等待多久后又执行，例如`@PerfTest(invocations = 500, threads = 100, timer = RandomTimer.class, timerParams = {200, 300})` ，每执行完一次后，等待200~300ms继续执行下一次.
5. 可能测试用例刚开始启动的时候，响应时间比较慢，需要运行一段时间后才能响应速度才会变快，这时我们就需要去掉前一段时间。这时我们就可以用到ContiPerf的rampUp和warmUp参数，例如`@PerfTest(threads = 5, duration = 50000, rampUp = 2000)` ，启动时会先起一个线程，然后每个2000ms起一线程，到8000ms时5个线程同时执行，那么这个测试实际执行了58s，这时我们要去掉前8s，那么就要在注释中加入warmUp，即`@PerfTest(threads = 5, duration = 50000, rampUp = 2000, warmUp = 8000)`，那么统计结果的时候会去掉预热的8s.
6. 也可以使用@Required指定性能要求
`@Required(max = 1000, average = 800, totalTime = 10000)`  要求每次执行的最长时间不能超过1s，平均时间不能超过0.8s，总时间不能超过10s;
`@Required(percentile90 = 5000)`要求90%的测试不超过5s;
`@Required(percentiles = "70:300,95:550")`要求70%的测试不超过300ms，95%的测试不超过550ms;


##### 折线图

文件目录`target/contiperf-report/index.html` 
![Image text](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/ContiPerf_Report.png)