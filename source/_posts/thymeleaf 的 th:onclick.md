---
title: 关于th:onclick的坑
date: 2019/09/22 12:15:25
updated: 2019/09/22 12:15:25
tags:
- java
- thymeleaf
---

### th:onclick的写法

1. 以前写法(请放弃)：

   方式一：

   ```html
   <button class="btn" th:onclick="'download(\'' + ${bean.filePath} + '\');'">下载</button>
   ```

   <!-- more -->

   方式二：

   ```html
   <button class="btn" th:onclick="'download(' + ${bean.filePath} + ');'">下载</button>
   ```

   方式三：

   ```html
   <button th:onclick="|download(${bean.filePath} )|">下载</button>
   ```

2. 最新的写法：

   ```html
   <button class="btn" th:onclick="download([[${bean.filePath}]]);">下载</button>
   ```

   