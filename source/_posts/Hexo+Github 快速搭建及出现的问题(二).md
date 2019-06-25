---
title: Hexo+Github 快速搭建及出现的问题(二)
date: 2017-03-07 09:49:08
tags: 
---

　　/(ㄒoㄒ)/真的是太久没更新了，主要是因为我的台式机电脑主板bios更新后，不支持es的u，而我的东西都放在三星的M2上，M2固态只有比较新的主板才支持，我几台电脑都没有M2接口。为了更新文章才买了个新u，是不是觉得很赞！

　　这次主要是讲解如何写文章并提交到Github上进行访问。

## 1.如何写文章
　　写文章的命令是：

    $ hexo new "文章的名字"

　　然后在hexo文件夹 `..\source\_posts`下就可以看到.md后缀的文件了，打开自己用markdown编辑（如果问我markdown怎么打开怎么写，我用小拳头锤你哦( ‵o′)）。


## 2.创建SSH Key
　　这个Key是作为提交到Github的凭证，步骤如下：
　　在hexo文件夹右键打开Git Bash Here，出现命令框，输入命令，生成一个SSH Key。

    $ ssh-keygen -t rsa -C "你的邮箱@你的邮箱.com"
    Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa): <回车就好>
    Enter passphrase (empty for no passphrase): <输入加密串>
    Enter same passphrase again: <再次输入加密串>

PS：在回车中会提示你输入一个密码，这个密码会在你提交项目时使用，如果为空的话提交项目时则不用输入。这个设置是防止别人往你的项目里提交内容。


## 3.添加SSH Key到Github
win8系统以下：打开本地`C:\Documents and Settings\Administrator.ssh\id_rsa.pub`文件。
win10系统：打开本地`C:\Users\你的当前用户\.ssh\id_rsa.pub`文件。
<br>　　登陆Github，点击的你头像——Settings——SSH and GPG keys——New SSH key。填写Title的名称随便填，然后把`id_rsa.pub`文件的key复制进去保存。
![ssh_add_key][1]

<br>接下来测试一下这个ssh key是否能连通：在命令行输入：

    $ ssh -T git@github.com
出现successfully就表示连接成功了
![ssh_test][2]
<br>
## 4.创建Github仓库并提交
　　我们Github，然后新建一个仓库，名称必须是：XXXX.github.io。其中XXXX是你自己的名字，以后输入这个就是首页了。
　　打开刚才你创建Github仓库，复制这个地址（要把https变成ssh哦）。
![ssh_url][3]
<br>在hexo文件下，打开`_config.yml`文件，找到deploy—repo那里：把地址粘贴进去。
![hexo_deploy][4]

<br>这时候，我们在命令行，输入：

    $ hexo clean
    $ hexo d -g
　　第一条命令是安装hexo
　　第二条命令是初始化生成静态文件并提交
<br>然后出现一堆东西，到最后，有`INFO  Deploy done: git`的提示，就表示完成啦~\(≧▽≦)/~
![deploy_done][5]


然后你直接输入XXXX.github.io，是不是就看到网站啦，很神奇是不是，赶紧去试试吧。<br><br>



 - 不过我知道肯定会出现各种问题，如有问题留言我看得到的话就回，(*^__^*)



[1]: http://lixin.piaozu.com.cn/ssh_add_key.png
[2]: http://lixin.piaozu.com.cn/ssh_test.png
[3]: http://lixin.piaozu.com.cn/ssh.png
[4]: http://lixin.piaozu.com.cn/hexo_deploy.png
[5]: http://lixin.piaozu.com.cn/deploy_done.png
