---
title: Hexo+Github 快速搭建及出现的问题
date: 2016-09-21 12:21:09
tags: 
---

本人使用的是win10系统，以下已windows为例子。

## 1.安装Git
　　去[Git官网](https://git-for-windows.github.io/)下载。安装过程一路默认即可，安装完后，鼠标右键菜单会多出两个Git按钮，Git GUI Here，Git Bash Here。
    
## 2.安装Node.js
　　同上，到[Node.js](https://nodejs.org/)下载。也是一路默认便可。
    
## 3.创建Hexo文件夹
　　在电脑上盘符新建一个文件夹，文件名夹名称用英文，比如我在D盘下创建MyHexo，进入这个空文件夹里面，鼠标右键，选择**Git Bash Here**，这时候就出现一个类似cmd的界面。
## 4.安装并初始化Hexo
　　因为Hexo是一个开源的静态博客生成器,用node.js开发，所以要使用Node.js来构建，输入Node.js的npm命令。

    $npm install -g hexo
    $hexo init
    $npm install
　　第一条命令是安装hexo
　　第二条命令是初始化hexo
　　第三天命令是安装依赖包
PS：貌似在初始化hexo的时候，我好像看到了dependencies succeed字样，那么，就不需要第三条命令，自己做的时候测试一下吧。
## 5.启动本地博客
　　上面命令已经把博客搭建好了，下面就来启动本地博客。

    $hexo s -g
　　这时候浏览器输入`localhost:4000`就会显示出Hexo默认博客了。
PS：Git命令界面显示Hexo is running，如果按Ctrl+C停止服务，那么刷新浏览器出现的默认Hexon博客就会无法链接。


　　未完待续。。。（后续会补一些图片）

