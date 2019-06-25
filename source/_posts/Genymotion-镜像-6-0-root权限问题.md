---
title: Genymotion 镜像 6.0 root权限问题
date: 2016-11-02 19:08:50
tags:
---

很久没写文章，因为在自己写一个app，然后我在Genymotion增加6.0镜像后，装RE的时候，发现里面没有没有root了！
![此处输入图片的描述][1]

后来机智的我看到了这么一个方法：
 ![此处输入图片的描述][2]

<br/>

### 本方法，针对Genymotion 6.0镜像。

 1. 下载[工具包][3]：里面有`Genymotion-ARM-Translation.zip`和`UPDATE-SuperSU-v2.46.zip`。这两个东西。<br/><br/>
 2. 启动Genymotion 6.0 镜像，首先把`Genymotion-ARM-Translation.zip`拖到镜像中，安装完了，有提示说重启，重开镜像就行。然后`UPDATE-SuperSU-v2.46.zip`也是老样子拖入。

<br/>
PS：当你把文件拖进去的时候，会出现这个错误：
![此处输入图片的描述][4]
<br/>
这个错误有两种解决方法：
　　1，把工具包路径有中文。
　　2，打开压缩文件，然后返回上一层，再拖进去。
![此处输入图片的描述][5]
<br/><br/>

[1]: http://lixin.piaozu.com.cn/%E9%9C%87%E6%83%8A.jpg
[2]: http://lixin.piaozu.com.cn/%E5%BE%97%E6%84%8F.jpeg
[3]: http://pan.baidu.com/s/1nvk5EXV
[4]: http://lixin.piaozu.com.cn/%E9%94%99%E8%AF%AF.png
[5]: http://lixin.piaozu.com.cn/%E5%8E%8B%E7%BC%A9.jpg