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
       threshold : 50,
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
                     //置为无数据，防止循环加载
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

   `me.resetload();`必须重置，所以用ajax加载数据的话就直接放在complete方法里重置就好。

   `threshold`：控制加载的距离，有些手机滚动界面和屏幕界面差距很大，可以通过增大该值来解决不加载数据的问题。

3. 参数列表 

   |    参数    |     说明     | 默认值                                                       | 可填值      |
   | :--------: | :----------: | :----------------------------------------------------------- | :---------- |
   | scrollArea |   滑动区域   | 绑定元素自身                                                 | window      |
   |   domUp    |   上方DOM    | {<br/>domClass : 'dropload-up',<br/>domRefresh : '&lt;div class="dropload-refresh"&gt;↓下拉刷新&lt;/div&gt;',<br/>domUpdate  : '&lt;div class="dropload-update"&gt;↑释放更新&lt;/div&gt;',<br/>domLoad : '&lt;div class="dropload-load"&gt;○加载中...&lt;/div&gt;'<br/>} | 数组        |
   |  domDown   |   下方DOM    | {<br/>domClass : 'dropload-down',<br/>domRefresh : '&lt;div class="dropload-refresh"&gt;↑上拉加载更多&lt;/div&gt;',<br/>domLoad : '&lt;div class="dropload-load"&gt;○加载中...&lt;/div&gt;',<br/>domNoData : '&lt;div class="dropload-noData"&gt;暂无数据&lt;/div&gt;'<br/>} | 数组        |
   |  autoLoad  |   自动加载   | true                                                         | true和false |
   |  distance  |   拉动距离   | 50                                                           | 数字        |
   | threshold  | 提前加载距离 | 加载区高度2/3                                                | 正整数      |
   |  loadUpFn  | 上方function | 空                                                           |             |
   | loadDownFn | 下方function | 空                                                           |             |

4. 已知问题

    由于部分Android中UC和QQ浏览器头部有地址栏，并且一开始滑动页面隐藏地址栏时，无法触发scroll和resize，所以会导致部分情况无法使用。解决方案1：增加distance距离，例如DEMO2中distance:50；解决方案2：加`meta`使之全屏显示 

    ```javascript
   <meta name="full-screen" content="yes">
   <meta name="x5-fullscreen" content="true">
    ```

5. 使用样例

   这是一个工具类，其中mobileAjax其实就是ajax，只是对ajax做了一层封装。

   ```javascript
   $.searchScroll = function(option) {
   			var defaults = {
   				url: "",//必传
   				boxId: '',//列表ul 必传
   				templateId: '',//模板id 必传
   				handleDataFunction: '',//处理前后台数据转换的函数；function (data) {}, 必传
   				databoxId: '',//外层div 必传 默认为 .data-box
   				searchTotalId: '',//显示总数位置 必传 默认为 .search-result-title
   				tempPage: 0,
   				queryData: {},
   				domDown: {},
   				scrollArea: '',
   				noDataClass: '',
   			};
   			//未知，定义成局部变量，滚动方法就会有问题。
   			searchOpt = $.extend(true, defaults, {
   				databoxId: '.data-box',
   				searchTotalId: '.search-result-title',
   				queryData: {
   					pageNum: 0,
   					pageSize: 5,
   					reasonable: false,
   				},
   				domDown: {
   					domClass: 'dropload-down',
   					domRefresh: '<div class="dropload-refresh" >上拉加载更多</div>',
   					domLoad: '<div class="dropload-load" ><span class="loading"></span>加载中...</div>',
   					domNoData: '<p class="search-result-bottom">我是有底线的</p>'
   				},
   				scrollArea: window,
   				noDataClass: 'no-data',
   			}, option);
   
   			if ($.isEmptyObject(searchOpt.boxId) || $.isEmptyObject(searchOpt.templateId) || $.isEmptyObject(searchOpt.url)) {
   				$.toast("必输参数不能为空", 1000);
   				return;
   			}
   			// if (!$.isFunction(searchOpt.handleDataFunction)) {
   			// 	$.toast("[handleDataFunction]入参必须为函数", 1000);
   			// 	return;
   			// }
   			$(searchOpt.boxId).empty();
   			$(".dropload-down").remove();
   			$(searchOpt.databoxId).removeClass('no-data');
   			$(searchOpt.databoxId).dropload({
   				scrollArea: searchOpt.scrollArea,
   				threshold: 120,
   				domDown: searchOpt.domDown,
   				loadDownFn: function (me) {
   					$.extend(true, searchOpt.queryData, {pageNum: ++searchOpt.tempPage});
   					console.log(searchOpt.queryData);
   					$.mobileAjax({
   						url: searchOpt.url,
   						data: searchOpt.queryData,
   						isHandleSuccess: false,
   						success: function (result) {
   							if ($.isEmptyObject(result)) {
   								me.lock();
   								me.noData();
   								$(".dropload-down").remove();
   								return;
   							}
   							if (result.code != 0) {
   								$.toast(result.code == 1001 ? result.msg : "请求失败", 1000);
   								me.lock();
   								me.noData();
   								return;
   							}
   							var data = result.data;
   
   							var total = 0;
   							if (data.hasOwnProperty("total")) {
   								total = data.total;
   							}
   							var rows = [];
   							if (data.hasOwnProperty("rows")) {
   								rows = data.rows;
   							} else {
   								// $.toast("返回数据格式有误", 1000);
   								// me.lock();
   								// return;
   								rows = data;
   							}
   							$(searchOpt.searchTotalId).text('为您搜索到' + total + '个结果');
   							//第一页没有结果，则直接显示无数据
   							if ($.isEmptyObject(rows) && searchOpt.tempPage == 1) {
   								$(searchOpt.databoxId).addClass(searchOpt.noDataClass);
   								$(".dropload-down").remove();
   								me.noData();
   								me.lock();
   								return;
   							}
   							//第二页没有结果，才显示 【有底线】
   							if ($.isEmptyObject(rows)) {
   								me.noData();
   								me.lock();
   								return;
   							}
   							var itemsSearch = rows;
   							//无重新处理数据的函数，则对数据不进行处理。
   							if ($.isFunction(searchOpt.handleDataFunction)) {
   								itemsSearch = searchOpt.handleDataFunction(rows);
   							}
   
   							$(searchOpt.boxId).append($.getTemplateHtml(itemsSearch, searchOpt.templateId));
   						},
   						complete: function (a, b) {
   							me.resetload();
   						},
   					});
   				}
   			});
   
   
   		},
   ```

### 总结

在刚入手这个插件的时候确实遇到蛮多问题，而仔细阅读了api文档后解决了不少困难，再结合github上的例子，算是完美入门了这个插件。虽然插件简单，但是我们还是要用心阅读api文档。