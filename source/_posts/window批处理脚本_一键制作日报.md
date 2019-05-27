---
title: 一键制作日报
date: 2019/05/23 12:50:25
updated: 2019/05/23 12:50:25
---

#### window 批处理命令

##### 背景
公司规定每天都要写日报，本来写日报就已经很烦了，可在写日报之前，还要一堆svn的操作就更烦了，所以就想着怎么能减少一点劳力呢。由于之前用linux写过自动化打包脚本，自然而然的就想到了window批处理命令。网上一查还真的可以各种操作。
<!--more-->
##### 代码思路
1. 首先更新svn。
2. 查看当前日期的文件夹是否已经存在了，不存在就创建一个文件夹。
3. 判断当前日期中是否已经存在日报，存在的话直接提交，并跳转到结束。
4. 不存在就复制昨天的日报到今天的日报文件夹中并且重命名。并且打开日报。
5. 命令行暂停，等待填写日报。
6. 提交svn，判断是提交整个文件夹，还是只需提交一个日报文件。
7. 结束流程。
代码优化过几次，主要有以下几个原因：
1. 定时任务中执行脚本的时候，当前目录可能并不是脚本所在目录，所以需要通过`%~dp0`来获取到当前脚本的目录，并切换过来。
2. 周末是不用写日报的，所以应该过滤掉周末（暂时不考虑法定假期）。
3. 周一过来复制文件的目录是前三天的，而不是前一天的。
##### 完整的脚本
脚本需要用电脑自动编辑器另存为ANSI编码格式，不然中文会乱码。
```dos
@echo off
rem 改成自己的名字
set userName=李林瑞
set fileName=信联征信_技术部_工作周报_%userName%_

rem 判断当前时间是否为工作日，法定假期除外
set workDate=%date:~-3%
if %workDate%==周六 goto endparms
if %workDate%==周日 goto endparms

rem 获取当前脚本目录
set currentPath=%~dp0
echo 当前脚本目录 : %currentPath%
cd /d %currentPath%

rem 获取当前日期20190522
set CURRENT_DATE=%date:~0,4%%date:~5,2%%date:~8,2%

rem 获取当前日期2019-05-22
set fileToday=%date:~0,4%-%date:~5,2%-%date:~8,2%

if %workDate%==周一 (set DaysAgo=3) else (set DaysAgo=1)
>"%temp%\MyDate.vbs" echo LastDate=date()-%DaysAgo%
>>"%temp%\MyDate.vbs" echo FmtDate=right(year(LastDate),4) ^& right("0" ^& month(LastDate),2) ^& right("0" ^& day(LastDate),2)
>>"%temp%\MyDate.vbs" echo wscript.echo FmtDate
for /f %%a in ('cscript /nologo "%temp%\MyDate.vbs"') do (set DstDate=%%a)

rem 获取昨天日期2019-05-21
set fileYesterday=%DstDate:~0,4%-%DstDate:~4,2%-%DstDate:~6,2%

rem 获取昨天日期20190521
set YESTERDAY_DATE=%DstDate:~0,4%%DstDate:~4,2%%DstDate:~6,2%

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
	copy %oldFile% %newFile%
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
:endparms
```
##### windows设置定时任务
1. 右键计算机，选择【管理】，打开【计算机管理】界面；或者直接输入命令`compmgmt.msc`。
2. 选择【任务计划程序】> 【任务计划程序】，点击创建基本任务。
3. 根据引导填写相应的信息创建基本任务
![Image text](https://raw.githubusercontent.com/lralin/TheFirst/master/markdown_img/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%AE%A1%E7%90%86-%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1.jpg)