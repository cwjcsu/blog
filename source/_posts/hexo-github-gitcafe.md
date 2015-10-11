title: hexo建立github,gitcafe博客并实时同步的要点
date: 2015-10-11 13:40:51
tags:
- blogs
- hexo
- github
categories:
- 工具
description:  把hexo博客的源码和生成的页面实时同步到github和gitcafe。
---
把hexo博客的源码和生成的页面实时同步到github和gitcafe。
<!--more-->

用搜索引擎搜索”github 博客”等关键字会出现大量很好的文章教小白一步步搭建。我这里列出一些关键点，希望可以让你少走弯路。这篇博客的markdown源代码在：https://gitcafe.com/cwjcsu/cwjcsu/blob/master/source/_posts/hexo-github-gitcafe.md
其他涉及的源码在同一个仓库可以找到。
 
## 1， 不一定要购买域名
很多文章都有介绍购买域名，并在根目录下配置CName文件，其实不一定要购买的。Github会给每个用户一个二级域名:cwjcsu.github.io。这个二级域名下，你可以定制样式、404页面等等，记住最重要的一点：你创建的github的仓库名称必须是cwjcsu.github.io,这样你只要在master分支上 仓库根目录push一个index.html，这个页面就可以通过 http://cwjcsu.github.io 访问到。cwjcsu是我的github用户名，实际操作中替换成你的即可。
 
## 2，使用gh_pages分支创建的页面
在github上，你可以为你的任何仓库添加一个网站，你只需要：
1. 把网站的页面push到这个仓库的gh_pages分支；（github有向导可以指引你自动创建这个分支）
2. 通过http://username.github.io/reponame 进行访问（即`http://博客地址/仓库名称/`）。我的github博客地址是：[`http://cwjcsu.github.io/`](http://cwjcsu.github.io/)，而joutable是我一个开源项目的仓库名称，它的页面可以通过http://cwjcsu.github.io/joutable 访问到
 
## 3，举例说明
我在github上面的首页是：https://github.com/cwjcsu
我在github上的博客源码仓库是：https://github.com/cwjcsu/cwjcsu.github.io
我在github上博客首页：http://cwjcsu.github.io
我的一个开源项目joutable仓库是：https://github.com/cwjcsu/joutable ，有两个分支，一个是master放置开源项目源码，一个是gh_pages放置项目介绍页面可以通过http://cwjcsu.github.io/joutable/ 访问
 
## 4，hexo搭建
hexo主页：https://hexo.io/
经过试用，hexo用来写博客真是不二选择（配合[sublime Text3](http://www.sublimetext.com/3) +[Markdown Editing](http://www.cnblogs.com/IPrograming/p/Sublime-markdown-editor.html),），具有下面的优势：
1. 使用markdown，完美支持github-flavored-markdown
2. 实时本地预览，（#hexo s 创建一个本地http-server在本地实时预览你的博客网站）
3. 大量丰富的主题模版(https://hexo.io/themes/)
4. 支持Tex语法(通过mathjax:https://www.mathjax.org )
5. 一键部署到多个站点（这个是我自己写的git脚本，下面有介绍）

使用hexo需要安装nodejs，npm，以及其他的依赖工具，网上教程不少，本文不赘述（遇到问题可以给我留言），不过特别提醒以下几点：
1. .yml 配置文件采用缩进进行分开，key和value之间至少要有一个空格；
2. 如果hexo生成的html里面有乱码，那是对应的源文件没有用UTF-8保存，你可以使用nodepadd++或者记事本把他们保存为UTF-8，然后重新生成即可；
3. 使用`<!--more-->`用来分割摘要和正文，上面部分是摘要，会出现在主页。同wordpress。
4. description:xxx 会生成网页的description描述:`<meta property="og:description" content="xxx">`这是SEO需要注意的地方。
 
### 推荐几篇好文章：
hexo搭建博客：http://www.cnblogs.com/zhcncn/p/4097881.html
hexo的Jacman主题：https://github.com/wuchong/jacman
hexo配置介绍：https://hexo.io/docs/configuration.html
markdown大全：http://cwjcsu.gitcafe.io/2015/09/26/markdown-learning/ 
 
## 5，hexo部署
在hexo配置文件_config.yml 有个deploy的配置项目用来配置git仓库，注意type需设置为git，是hexo3中的类型，需要安装：hexo-deployer-git：
```
npm install hexo-deployer-git --save
```
网上的教程大多是hexo2的，很多文章没有指出这个区别。

#hexo deploy 可以一键部署到github仓库，但是我需要部署到不同仓库，所以没有采用hexo的自动部署，二是写了一个脚本：
在hexo生成的博客根目录cwjcsu.github.io下有个脚本：up.sh

```
#!/bin/bash

git commit -am "$1" 
git push github master:master 
git push gitcafe master:gitcafe-pages 
```
于是，可以通过下面命令一键把生成的博客内容更新到github和gitcafe了。
```
#./up.sh "commit comment" 
```
 下面是我的git配置文件：cwjcsu.github.io/.git/config

```
[core]
    repositoryformatversion = 0
    filemode = false
    bare = false
    logallrefupdates = true
    symlinks = false
    ignorecase = true
    hideDotFiles = dotGitOnly

[remote "gitcafe"]
    url = git@gitcafe.com:cwjcsu/cwjcsu.git
    fetch = +refs/heads/*:refs/remotes/gitcafe/*

[remote "github"]
    url = git@github.com:cwjcsu/cwjcsu.github.io.git
    fetch = +refs/heads/*:refs/remotes/github/*
    
[branch "master"]
    remote = github
    merge = refs/heads/master
```
 可以看到有两个remote：github和gitcafe，分别配置github和gitcafe上面我的博客所在的仓库地址。up.sh脚本中gitcafe的分支是gitcafe-pages而不是master，仓库名称也不是域名而是直接用户名。这是因为gitcafe博客与github博客略有不同，下面会介绍。
 为了避免每次push都提示你输入用户名和密码，你需要在github和gitcafe中添加你的公钥，具体操作本文不赘述。[user]部分我没有贴出来。

 ## 6,gitcafe博客
 与github不同的是，创建gitcafe的博客，**你只需要创建一个和你的用户名一样的仓库，然后把页面push到这个仓库的gitcafe-pages分支即可**，然后把源码push到这个分支的master。
 比如：
 博客所在仓库是：https://gitcafe.com/cwjcsu/cwjcsu
 博客源码的分支是master：https://gitcafe.com/cwjcsu/cwjcsu/tree/master
 博客页面分支是gitcafe-pages：https://gitcafe.com/cwjcsu/cwjcsu/tree/gitcafe-pages

 这两个分支与github上两个仓库代码是同一份，我又写了个脚本，用来实时push博客源码到两个仓库：blog/up.sh: (blog是博客源码所在目录)
```
#!/bin/bash

git commit -am "$1" 
git push github master:master 
git push gitcafe master:master
```
blog/.git/config:
```
[core]
    repositoryformatversion = 0
    filemode = false
    bare = false
    logallrefupdates = true
    symlinks = false
    ignorecase = true
    hideDotFiles = dotGitOnly

[remote "gitcafe"]
    url = git@gitcafe.com:cwjcsu/cwjcsu.git
    fetch = +refs/heads/*:refs/remotes/gitcafe/*

[remote "github"]
    url = git@github.com:cwjcsu/blog.git
    fetch = +refs/heads/*:refs/remotes/github/*
    
[branch "master"]
    remote = github
    merge = refs/heads/master
```

[user]部分我没有贴出来。

## 7,配置评论系统和百度统计
我使用了国内很火的一个评论系统：[多说](http://duoshuo.com/),注册、添加站点后，只需要把ID设置到hexo主题的_config.yml文件里面即可（注意不是hexo的配置文件而是themes/jacman/_config.yml，jacman是我使用的一个主题）。添加百度统计网上教程很多不赘述。


 
 