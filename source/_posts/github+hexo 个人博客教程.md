---
title: Github+Hexo 个人博客搭建教程
date: 2018-02-01
tags: github
---
##### 1.下载Node.js安装文件：
[Window Installer 64-bit](https://nodejs.org/dist/v4.2.3/node-v4.2.3-x64.msi)<br/>
保持默认设置即可，一路Next，安装很快就结束了。 然后我们检查一下是不是要求的组件都安装好了，在cmd界面输入：
```
node -v
v.0.12.1   #表示成功
npm -v
2.5.1   #表示成功
```
<!--more-->
linux 系统安装教程 https://nodejs.org/en/download/package-manager/#arch-linux  

centos 安装命令
```
curl --silent --location https://rpm.nodesource.com/setup_8.x | sudo bash -
```
```
sudo yum -y install nodejs
```
Optional: install build tools
To compile and install native addons from npm you may also need to install build tools:
```
sudo yum install gcc-c++ make
# or: sudo yum groupinstall 'Development Tools'
```


##### 2.安装git
##### 3.安装Hexo
在自己认为合适的地方创建一个文件夹，这里我以E：/hexo 为例子讲解，首先在E盘目录下创建Hexo文件夹，并在命令行的窗口进入到该目录
![image](https://github.com/lralin/TheFirst/raw/master/markdown_img/hexo-cd_image.jpg)  
在命令行输入：
```
npm install hexo-cli -g
```
然后你将看到：  
![image](https://github.com/lralin/TheFirst/raw/master/markdown_img/hexo-cli_image.jpg)
可能你会看到一个WARN，但是不用担心，这不会影响你的正常使用。 
在命令行中输入：
```
hexo -v
```
如果你看到了如图文字，则说明已经安装成功了。

![image](https://github.com/lralin/TheFirst/raw/master/markdown_img/hexo-success_image.jpg)

##### 4.hexo的相关配置
**初始化Hexo**  
```
hexo init blog
cd blog
```
**首次体验Hexo**  
继续操作，同样是在命令行中，输入：  
```
# 首次操作可以不执行，界面已经编译好了。
hexo g
```
![image](https://github.com/lralin/TheFirst/raw/master/markdown_img/hexo-g_image.jpg)  
然后输入：
```
hexo s
```
然后会提示：
```
INFO Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.
```
在浏览器中打开http://localhost:4000/  
[主题地址](https://github.com/hexojs/hexo/wiki/Themes)
##### 5.配置Github

**Git 免密**  
（1）打开git bash，在用户主目录下运行：`ssh-keygen -t rsa -C "youremail@example.com"`  把其中的邮件地址换成自己的邮件地址，然后一路回车  
（2）最后完成后，会在用户主目录下生成.ssh目录，执行`cd ~/.ssh/`，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH key密钥对，id_rsa是私钥，千万不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人,`cat id_rsa.pub`。  
（3）登陆GitHub，打开「Settings」->「SSH and GPG keys」，然后点击「new SSH key」，填上Title，在Key文本框里粘贴公钥id_rsa.pub文件的内容（千万不要粘贴成私钥了！），最后点击「Add SSH Key」，你就应该看到已经添加的Key。 

**hexo 部署编译文件到github**  
修改_config.yml文件，来建立关联，命令：

```
vim _config.yml
```
翻到最下面，改成我这样子的
```
deploy:

type: git

repo: https://github.com/username/username.github.io.git

branch: master
```
然后执行命令：
```
npm install hexo-deployer-git --save
```
配置好git ssh后
执行配置命令：
```
# 通过命令推送编译好的源码到github上面。然后就可以直接通过youname.github.io访问了。
hexo deploy
```
**部署步骤**

每次部署的步骤，可按以下三步来进行。
```
hexo clean

hexo generate

hexo deploy
```
**常用命令**：
```
hexo new"postName" # 新建文章

hexo new page"pageName" # 新建页面

hexo generate # 生成静态页面至public目录

hexo server # 本来浏览 可以简写成 hexo s 开启预览访问端口（默认端口4000，'ctrl + c'关闭server）

hexo deploy # 将.deploy目录部署到GitHub

hexo help # 查看帮助

hexo version # 查看Hexo的版本   
```
**备份源码**  

把Hexo的源码备份到Github分支里面，思路就是上传到分支里存储，修改本地的时候先上传存储，再发布。更换电脑的时候再下载下来源文件打开git-bash
```
$ git init
$ git remote add origin git@github.com:username/username.github.io.git		
$ git add .
$ git commit -m "blog"
$ git push origin master:Hexo-Blog
```
现在你会发现github你的博客仓库已经有了一个新分支Hexo-Blog，我们的备份工作完成。  
**换一台电脑之后处理Hexo**
1. 使用`git clone git@github.com:username/username.github.io.git`拷贝仓库（分支为Hexo-blog）；
2. 在本地新拷贝的http://lralin.github.io文件夹下通过Git bash依次执行下列指令：`npm install hexo`、`npm install`、`npm install hexo-deployer-git --save`（记得，不需要`hexo init`这条指令）。  
 
**添加网易云音乐**
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86   
    src="http://music.163.com/outchain/player?type=2&id=25706282&auto=0&height=66">  
</iframe>  
