title: 我的博客
date: 2015-09-25 19:14:17
tags:
- blogs
- hexo
- jacman

# 搭建github博客之前必须了解的东西
用搜索引擎搜索"github 博客"等关键字会出现大量很好的文章教小白一步步搭建。我这里列出一些容易走弯路指引。
## 不一定要注册域名
很多文章都有介绍购买域名，并在根目录下配置CName文件，其实不一定要购买的。Github会给每个用户一个二级域名:`<username>.github.io`。这个二级域名下，你可以定制样式、404页面等等，记住最重要的一点：**你创建的github的Repository name必须是`<username>.github.io`**,这样你只要在**master**分支上仓库根目录上push一个`index.html`，这个文件就可以通过`http://<username>.github.io`访问到。
`<username>`是你的github用户名。

## 仓库`<repoName>`的Github Pages可以通过`<username>.github.io/<repoName>`访问
### 手工创建
很多博客都有介绍怎么建立github pages，你可以给你的任意仓库建立github pages，只是你必须把页面push到一个特定名字的分支：**gh_pages**。这样，这个分支里面的页面就可以通过`<username>.github.io/<repoName>`访问，`<repoName>`是这个仓库的名称。

### github向导创建
使用github的向导创建Github Pages，你可以选择它提供的模版，github会在后台给你创建**gh_pages**分支，并把模版所需的css、JavaScript、images、fonts全部上传到这个分支里面，和你手工创建没有区别。
github向导创建的index.html等页面，对css等资源的引用采用的的相对路径，所以通过`<username>.github.io/<repoName>/`访问这个Pages不会出现资源的404错误，即使你修改了`<username>`。当然，有些链接是采用绝对路径，比如:
```
<li class="downloads"><a href="https://github.com/<username>/<repoName>/zipball/master">ZIP</a></li>
```
这个链接是下载当前仓库master分支的zip文件，绝对路径里面包含`<username>`和`<repoName>`，所以如果你修改了用户名`<usernmae>`或者当前仓库的名称`<repoName>`，那么这个链接就失效了。

## Github Pages也可以用[hexo](https://hexo.io)等进行定制
网上很多文章都说用**hexo+特定主题**创建github+pages，会出现资源文件404错误，然后说可以去买个域名，创建CName映射文件解决问题。我在初次搭建github博客的时候，差点就下单买域名了,其实是不必要的。
在hexo的配置文件`_config.yml`里面修改配置:
```
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: cwjcsu.github.io
root: /<repoName>/
```
那么hexo就会在生成页面的时候，给所有导入的资源加上`/<repoName>/`，从而可以正确访问到仓库`<repoName>`里面的资源文件，这样就会显示正常。

# Tips about hexo + jacman

1. .yml配置项冒号和值之间要有空格
1. jacman主题的内置分享功能url没有用`http://`开头，导致分享不成功
    我修改了一下代码：themes/jacman/layout/_partial/after_footer.ejs:(大概第86行左右)
    ```
     var $this = $('.share'),
      url = $this.attr('data-url');
      if(url.indexOf('http://')<0){
         url = 'http://' + url;
      }
      var encodedUrl = encodeURIComponent(url),
      title = $this.attr('data-title'),
      tsina = $this.attr('data-tsina'),
      description = $this.attr('description');
    ```

1. 一些.md或者.html被hexo处理后出现中文乱码
    往往是.md或者.html文件使用的编码格式不是UTF-8，只需要把这些文件转换成UTF-8并保存就可以被hexo正常处理；

1. 每篇文章的description属性会被hexo处理成description的meta，即：
    ```
        title: hexo你的博客
        date: 2013-11-22 17:11:54
        categories: default
        tags: [hexo]
        description: 你对本页的描述
    ```

处理之后会在html页面的&lthead&gt标签内生成：
    ```
        <meta property="og:description" content="你对本页的描述"> 
    ```
这个描述一般会在搜索引擎展示你的页面时出现。（SEO范畴）
1. 使用hexo new page *pageName*会创建一个source/*pageName*目录，并在这里面创建一个index.md文件，这个页面可以通过`<username>.github.io/<pageName>`访问到。所以这个*pageName*不要与你的某个聚友Github Pages的仓库名称重复了。

1. 使用`<!--more-->`标签可以让标签以上的文本作为摘要，在博客列表中显示出来，点击Read More显示特定博客有内容。

