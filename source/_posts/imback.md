title: 我回来了
date: 2016-06-13 00:41:40
tags:
- blogs
- hexo
categories:
- Misc
description:  I'm back.我回来了。 
---

由于家里的电脑重装了系统，昨晚重新搭建了node+hexo环境。结果`hexo s`没有任何异常，但是无法通过http://localhost:4000 查看到任何文章。后来...
<!--more-->
后来用下面命令找到了原来是Foxit程序占用了这个端口。
```
C:\Users\Atlas>netstat -aon |findstr 4000
```

果断结束进程，然后去`msconfig`里面把Foxit相关的自启动服务全部停止。

你以为这就完了？

NO！！！
sublime text 3又闹脾气了，（这个版本[win 7 64 bit build 3114](https://download.sublimetext.com/Sublime%20Text%20Build%203114%20x64%20Setup.exe)），直接把系统卡成翔。

重装subl3=》打开md文件：卡死，=》重启；

卸载subl3的一个markdown组件的问题=》打开md文件，卡死=》重启；

使用portable版本，并且删除sublime的程序文件，打开md文件，OK了。

### 昨天去[digitalocean](https://cloud.digitalocean.com)充值了25美元，够用5个月了。

### 今天开始，整理代码并写博客。


