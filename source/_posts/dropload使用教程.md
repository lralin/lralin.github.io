---
title: dropload手机滚动下拉插件
date: 2019-12-22 20:35:25
updated: 2019-12-22 21:00:25
tags: 
- javascript
- 分页加载

---

### 前言

最新项目在开发一个手机端的页面，对于下拉加载数据是必须的。本想自己手写一个，但是本着不重复造轮子的想法，就像在GitHub上找找有没有比较好的插件。一查就发现了这个[dropload](https://github.com/ximan/dropload)，支持上拉和下拉加载数据。

<!--more-->

### 使用说明

1. 引入js和css

   ```java
   <link rel="stylesheet" href="../dist/dropload.css">
   <script src="../dist/dropload.min.js"></script>
   ```

2. 渲染数据

   ```javascript
   $('.element').dropload({
       scrollArea : window,
       loadDownFn : function(me){
           $.ajax({
               type: 'GET',
               url: 'json/more.json',
               dataType: 'json',
               success: function(data){
                   alert(data);
                   if($.isEmptyObject(data)){
                     //如果数据为空，则必须锁定
                     me.lock();
                     //致为无数据，防止循环加载
                     me.noData();
                   }
                   // 每次数据加载完，必须重置
                   me.resetload();
               },
               error: function(xhr, type){
                   alert('Ajax error!');
                   // 即使加载出错，也得重置
                   me.resetload();
               }
           });
       }
   });
   ```

   `me.resetload();`因为必须重置，如果是用ajax加载数据的话就直接放在complete方法里重置就好。

3. 参数列表

   |    参数    |     说明     | 默认值                                                       | 可填值                                                 |
   | :--------: | :----------: | :----------------------------------------------------------- | :----------------------------------------------------- |
   | scrollArea |   滑动区域   | 绑定元素自身                                                 | window                                                 |
   |   domUp    |   上方DOM    | {     <br />domClass : 'dropload-up',     <br />domRefresh : '<div class="dropload-refresh">↓下拉刷新</div>',     domUpdate : '<div class="dropload-update">↑释放更新</div>',     domLoad : '<div class="dropload-load">○加载中...</div>' <br />} | 数组                                                   |
   |  domDown   |   下方DOM    | {<br/>domClass : 'dropload-down',<br/>domRefresh : '<div class="dropload-refresh">↑上拉加载更多</div>',<br/>domLoad : '<div class="dropload-load">○加载中...</div>',<br/>domNoData : '<div class="dropload-noData">暂无数据</div>'<br/>} | 数组                                                   |
   |  autoLoad  |   自动加载   | true                                                         | true和false                                            |
   |  distance  |   拉动距离   | 50                                                           | 数字                                                   |
   | threshold  | 提前加载距离 | 加载区高度2/3                                                | 正整数                                                 |
   |  loadUpFn  | 上方function | 空                                                           | function(me){<br/>//你的代码<br/>me.resetload();<br/>} |
   | loadDownFn | 下方function | 空                                                           | function(me){<br/>//你的代码<br/>me.resetload();<br/>} |

   