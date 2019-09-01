---
title: 多线程获取任务完成
date: 2019-08-31 13:57:25
updated: 2019-08-31 17:57:25
tags: 
- java
- 多线程
---

### 前言

CountDownLatch：CountDown英文解释为倒数，Latch是锁上，门闩的意思。在开发时，当我们有三个并发任务同时去执行，然后在下一步需要等待它们都执行完成才能继续执行，实现这个现在想想大概有几种方式，Thread.join等待线程，使用wait()notify()，Executor.invokeAll、然后对各个Future调用get等。而我们的CountDownLatch类就是专门用来控制这种情况的。初始化的时候创建一个倒计时次数count，通过调用countDown()方法对count的次数进行减1，当count次数为0的时候，调用await()的等待的线程会被释放。

### 代码

1. 创建线程池

   ```java
   @EnableAsync
   @Configuration
   public class ThreadPoolTask {
       @Bean("taskExecutor")
       public Executor taskExecutor() {
           ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
           executor.setCorePoolSize(10);
           executor.setMaxPoolSize(20);
           executor.setQueueCapacity(200);
           executor.setKeepAliveSeconds(60);
           executor.setThreadNamePrefix("taskExecutor-");
           //线程池对拒绝任务的处理策略：这里采用了CallerRunsPolicy策略，当线程池没有处理能力的时候，该策略会直接在 execute 方法的调用线程中运行被拒绝的任务；如果执行程序已关闭，则会丢弃该任务
           executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
           return executor;
       }
   }
   ```

2. 定义线程

   ```java
   @Component
   public class TaskTest {
       @Async("taskExecutor")
       public void doTask(CountDownLatch count) throws Exception {
       		int num = RandomUtils.nextInt()
           log.info("开始做任务:{}",num);
           long start = System.currentTimeMillis();
           Thread.sleep(num);
           long end = System.currentTimeMillis();
         	count.countDown();
           log.info("完成任务{}，耗时：{}毫秒",num,(end - start));
       }
   }
   ```

3. 测试

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)
   @SpringBootTest
   public class ApplicationTests {
       @Autowired
       private TaskTest task;
   
       @Test
       public void test() throws Exception {
           int runnerCount = 10;
         	//初始化计数次数。
           CountDownLatch count = new CountDownLatch(runnerCount);
         	log.info("线程池开始");
           for (int i = 0; i < runnerCount; i++) {
               task.doTask(count);
           }
         	//只有当task中所有的任务完成了，await方法才会被释放，不然会一直阻塞。
         	//当然也可以使用他的超时方法count.await(5, TimeUnit.SECONDS);当所有线程任务的执行完成时间，超过5分钟的时候，会超时释放。
         	count.await();
   	      log.info("线程池结束");
       }
   }
   ```

### 总结

CountDownLatch使用起来还是很简单的。在需要知道其他线程任务已经完成，才能继续执行的方法中有很大的作用。