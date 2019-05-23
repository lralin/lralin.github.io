---
title: 一键制作日报
date: 2019/05/23 12:50:25
updated: 2019/05/23 12:50:25
---

#### window 批处理命令

##### 背景
公司规定每天都要写日报，本来写日报就已经很烦了，可在写日报之前，还要一堆svn的操作就更烦了，所以就想着怎么能减少一点劳力呢。由于之前用linux写过自动化打包脚本，自然而然的就想到了window批处理命令。网上一查还真的可以各种操作。

##### 代码思路
1. 首先更新svn。
<!--more-->
2. 查看当前日期的文件夹是否已经存在了，不存在就创建一个文件夹。
3. 判断当前日期中是否已经存在日报，存在的话直接提交，并跳转到结束。
4. 不存在就复制昨天的日报到今天的日报文件夹中并且重命名。并且打开日报。
5. 命令行暂停，等待填写日报。
6. 提交svn，判断是提交整个文件夹，还是只需提交一个日报文件。
7. 结束流程。
##### 完整的脚本
```dos
@echo off
:: 改成自己的名字
set userName=lralin
set fileName=XXXXX_技术部_工作周报_%userName%_

rem 获取当前脚本目录
set currentPath=%~dp0
echo 当前脚本目录 : %currentPath%
cd /d %currentPath%

rem 获取昨天日期20190521
set YE=%date:~0,4%
set MO=%date:~5,2%
set DA=%date:~8,2%
set DG=1
set/a vY1=%YE% %% 400
set/a vY2=%YE% %% 4
set/a vY3=%YE% %% 100
if %vY1%==0 (set var=true) else (if %vY2%==0 (if %vY3%==0 (set var=false) else (set var=true)) else (set var=false))
set LY=%YE%
set LM=%MO%
if %MO:~0,1%==0 (set MO=%MO:~1,1%)
if %DA:~0,1%==0 (set DA=%DA:~1,1%)
if %DA% GTR %DG% (set/a LD=%DA%-%DG%) else (
if %MO%==1 (set/a LY=%YE%-1) & (set/a LM=12+%MO%-1) & (set/a LD=31+%DA%-%DG%) else (
set/a LM=%MO%-1
if %MO%==3 (if %var%==false (set/a LD=28+%DA%-%DG%) else (set/a LD=29+%DA%-%DG%))
for %%a in (2 4 6 8 9 11) do (if "%MO%"=="%%a" (set/a LD=31+%DA%-%DG%))
for %%b in (5 7 8 10 12) do (if "%MO%"=="%%b" (set/a LD=30+%DA%-%DG%))))
if %LM% LSS 10 set LM=0%LM:~-1%
if %LD% LSS 10 set LD=0%LD:~-1%
set YESTERDAY_DATE=%LY%%LM%%LD%

rem 获取当前日期20190522
set CURRENT_DATE=%date:~0,4%%date:~5,2%%date:~8,2%

rem 获取昨天日期2019-05-21
set fileYesterday=%LY%-%LM%-%LD%

rem 获取当前日期2019-05-22
set fileToday=%date:~0,4%-%date:~5,2%-%date:~8,2%

cd 日报集
rem 1.首先更新svn
svn update

set isExist=true 
echo %CURRENT_DATE%

rem 2.查看当前日期的文件夹是否已经存在了
if exist %CURRENT_DATE% (
	set isExist=true
	rem cd %CURRENT_DATE%
	echo "当前日期的文件夹已经存在"
) else (
	set isExist=false
	echo "当前日期的文件夹不存在,创建文件夹"
	mkdir %CURRENT_DATE%
)

set oldFile=%YESTERDAY_DATE%\%fileName%%fileYesterday%.xlsx
set newFile=%CURRENT_DATE%\%fileName%%fileToday%.xlsx
echo %oldFile%
echo %newFile%
rem 3.复制昨天的日报到今天的日报文件夹中并且重命名
if not exist %newFile% (
	echo 今天的日报不存在复制昨天的日报
	rem 复制文件
	copy %oldFile% %newFile%
	rem 打开文件
	start %newFile%
) else (
	echo 今天的日报已经存在，直接进行svn自动上传。
	cd %CURRENT_DATE%
	svn commit -m "每日周报上传-%userName%"
	goto noparms
)
echo 请填写今天的日报，修改完成后按下enter键进行svn自动上传。
Pause
echo 确认文件是否已经关闭!
Pause
echo %isExist%
if %isExist%==true (
	echo 文件夹已经存在，只需上传文件
	cd %CURRENT_DATE%
	svn add %fileName%%fileToday%.xlsx
	svn commit -m "每日周报上传-%userName%"
) else (
	echo 文件夹不存在，上传整个文件夹
	svn add %CURRENT_DATE%
	svn commit -m "每日周报上传-%userName%"
)
:noparms
echo 命令执行结束！！！！！！
Pause
```
##### windows设置定时任务
1. 右键计算机，选择【管理】，打开【计算机管理】界面；或者直接输入命令`compmgmt.msc`。
2. 选择【任务计划程序】> 【任务计划程序】，点击创建基本任务。
3. 根据引导填写相应的信息创建基本任务
![Image text](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%AE%A1%E7%90%86-%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1.jpg)