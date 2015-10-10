title: Markdown学习
date: 2015-09-26 15:01:19
tags:
- 工具
- markdown

categories: 
- 工具
description: 介绍了标准Markdown语法和GitHub Flavored Markdown (GFM)，并推荐了一款Markdown编辑器
---

本文介绍了标准Markdown语法和GitHub Flavored Markdown (GFM)，并推荐了一款Markdown编辑器
<!--more-->
# 标准Markdwon语法
**Markdown**是一种轻量级标记语言，它允许人们“使用易读易写的纯文本格式编写文档，然后转换成有效的HTML(或者pdf、docx)文档”。Markdown现在拥有非常多的扩展，而且标准化已经几乎不可能。有一个介绍标准化markdown的[网站](http://daringfireball.net/projects/markdown/)。里面的[Syntax](http://daringfireball.net/projects/markdown/syntax)详细介绍了标准markdown所有语法。

## 块元素(Block Elements)
### 文档标头（HEADERS）
有两种类型的标题：Setext-style headers  和 Atx-style headers。
（1），Setext-style headers 的markdwon代码：
```
This is H1
=
This is H1
---
```
其中等于号(=)和破折号(-)的个数没有限制（好像要超过2个）。（效果免了，否则我这篇博客会被撑乱）。
（2），Atx-style headers 示例：
```
# This is an H1
## This is an H2
###### This is an H6
```
注意#号后面要加空格，当前文档使用的就是这种格式标题。另外  atx-style 的头可以关闭：
```
# This is an H1 #
## This is an H2 ##
### This is an H6 ######
```

### 块引用(BLOCKQUOTES)
markdown使用 > 来做块引用（email-style quote）
```
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
```
效果：
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
> 
> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
> id sem consectetuer libero luctus adipiscing.

Blockquotes可以只在段首使用 > ,效果是一样的：
```
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
```

Blockquotes可以嵌套：
```
> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.
```
效果：
> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.

Blockquotes里面可以嵌套其他markdown元素，包括headers,lits,code blocks等等：
```
> #### This is a header.
> 
> 1.   This is the first list item.
> 2.   This is the second list item.
> 
> Here's some example code:
> 
>     return shell_exec("echo $input | $markdown_script");
```
效果如下：
> #### This is a header.
> 
> 1.   This is the first list item.
> 2.   This is the second list item.
> 
> Here's some example code:
> 
>     return shell_exec("echo $input | $markdown_script");

### 列表
（1），无序列表使用星号(*,asterisks)
```
* Red
* Green
```
等价于：
```
+ Red
+ Green
```
也等价于：
```
- Red
- Green
```
效果如下：
- Red
- Green
（2），有序列表使用数字：
```
1. Bird
2. McHale
```
可以写成：
```
1. Bird
1. McHale
```
甚至
```
8. Bird
3. McHale
```
总之，只要是数字加一个点即可。效果如下：
8. Bird
1. McHale
（3），列表默认都是从左边开始的，如果想加缩进，可以在前面加超过3个以上的空格。
```
*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
    Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
    
    viverra nec, fringilla in, laoreet vitae, risus.
*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
    Suspendisse id sem consectetuer libero luctus adipiscing.
```
效果是这样的：
*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
    Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
    
    viverra nec, fringilla in, laoreet vitae, risus.
*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
Suspendisse id sem consectetuer libero luctus adipiscing.

如果列表项之间有空行，markdown会给每一个生成的li元素创建一个p:
```
*   Bird

*   Magic
```
产生的DOM是:
```
<ul>
<li><p>Bird</p></li>
<li><p>Magic</p></li>
</ul>
```
（4），在列表之中使用blockquote，需要有缩进：
```
*   A list item with a blockquote:

    > This is a blockquote
    > inside a list item.
```
效果是这样的：
*   A list item with a blockquote:

    > This is a blockquote
    > inside a list item.

（5），列表项中放入代码块，代码库必须要缩进8个空格（或者2个tab）：

```
*   A list item with a code block:

        <code goes here>
```
效果：
*   A list item with a code block:

        <code goes here>

（6），因为一个段落有数字开头，从而创建了一个有序列表时，可以用下面的方式避免：
```
1986. What a great season.
```
写成：
```
1986\. What a great season.
```

1986\. What a great season.

### 代码块（CODE BLOCKS）

（1），html中代码块是被&lt;code&gt;或者&lt;pre&gt;等标签包裹的，在markdwon里面，只需要在代码库开头空一行，代码行缩进4个空格（一个tab）即可。
```
This is a normal paragraph:

    This is a code block.
```
markdown会生成：
```
<p>This is a normal paragraph:</p>

<pre><code>This is a code block.
</code></pre>
```
This is a normal paragraph:

    This is a code block.
生成的html会把每个代码行的移除4个空格（或者一个tab）：
代码：
```
Here is an example of AppleScript:

    tell application "Foo"
        beep
    end tell
```
会被解析成：
```
<p>Here is an example of AppleScript:</p>

<pre><code>tell application "Foo"
    beep
end tell
</code></pre>
```
代码块会一直继续，直到遇到一个没有缩进的行或者文档末尾。
（2），在代码块内部，这几个符号：&amp;、&lt; 、 &gt;都会被自动转义，比如：
```
    <div class="footer">
        &copy; 2004 Foo Corporation
    </div>
```
会变成：
```
<pre><code>&lt;div class="footer"&gt;
    &amp;copy; 2004 Foo Corporation
&lt;/div&gt;
</code></pre>
```
效果如下：
    <div class="footer">
        &copy; 2004 Foo Corporation
    </div>

其他markdwon字符在代码块中不会被处理，比如*。

### 水平规则（HORIZONTAL RULES）
可以通过在一行写3个以上的 来产生一条水平线
```
* * *

***

*****

- - -

---------------------------------------
```

效果如下：（只取了有一条）

* * *

## SPAN元素（SPAN ELEMENTS）
### 链接(Links)
markdwon支持两种类型的链接:*inline and reference*，两种链接都用中括号\[\]界定。
（1）inline链接：
```
This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.
```

效果是这样的：
This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.
引用本地资源（在同一台服务器上的）：

```
See my [About](/about/) page for details. 
```
See my [About](/about/) page for details. 

（2），引用连接(Reference-style)使用一个后置的中括号
```
This is [an example] [id] reference-style link.
```
效果是这样的：
This is [an example] [id] reference-style link.
然后文档中的任何其他地方可以这样定义被引用的链接：
```
[id]: http://example.com/  "Optional Title Here"
```
这个markdwon标签不会产生html，所以不会显示（链接的定义支队markdwon解析有效，不会生成html）。
[id]: http://example.com/  "Optional Title Here"
下面三种引用链接的定义方式等价：
```
[foo]: http://example.com/  "Optional Title Here"
[foo]: http://example.com/  'Optional Title Here'
[foo]: http://example.com/  (Optional Title Here)
```
其中\[id\]可以包含数字、字母、空格、标点符号，但是大小写不敏感。

（3），可以隐性定义引用链接：（链接的Label用作id）
```
I get 10 times more traffic from [Google][] than from
[Yahoo][] or [MSN][].

  [google]: http://google.com/        "Google"
  [yahoo]:  http://search.yahoo.com/  "Yahoo Search"
  [msn]:    http://search.msn.com/    "MSN Search"
```

效果如下：
I get 10 times more traffic from [Google][] than from
[Yahoo][] or [MSN][].

  [google]: http://google.com/        "Google"
  [yahoo]:  http://search.yahoo.com/  "Yahoo Search"
  [msn]:    http://search.msn.com/    "MSN Search"

### 强调（EMPHASIS）
markdown把用*和下划线_包裹的文本作为强调文本：
```
*单星*
_单下划线_
**双星**
__双下划线__
```
效果如下：
*单星*
_单下划线_
**双星**
__双下划线__
强调可以加载文字中间：
```
un*frigging*believable
```
效果是：
un*frigging*believable
要书写字面量*，则需要用反斜杠:
```
\*this text is surrounded by literal asterisks\*
```
\*this text is surrounded by literal asterisks\*

### 代码（CODE）
使用反引号（backtick,\`)来包裹一段代码：Use the `printf()` function.
```
Use the `printf()` function.
```
为了在代码中使用反引号字面量，可以用多个反引号包裹代码：
```
``There is a literal backtick (`) here.``
```
上面代码示例在我的markdwon源文件里面是用三个反引号包裹的，它产生html：
```
<p><code>There is a literal backtick (`) here.</code></p>
```

在code span中，这几个符号：&amp;、&lt; 、 &gt;都会被自动转义成（同代码块）
```
Please don't use any `<blink>` tags.
```
会被转义成：
```
<p>Please don't use any <code>&lt;blink&gt;</code> tags.</p>
```

### 图片（IMAGES）
markdwon中有两种图片的标签：*inline and reference*。
Inline图片语法是这样的：（注意惊叹号，中括号中指定图片的`alt`属性，小括号是图片的URL外加一个可选的`title`属性）
```
![Alt text](/path/to/img.jpg)
![Alt text](/path/to/img.jpg "Optional title")
```

Reference-style图片语法是这样的：（同样是惊叹号，中括号中指定图片的`alt`属性，又一个中括号指定引用id）
```
![Alt text][id]
```
图片引用这么写：
```
[id]: url/to/image  "Optional title attribute"
```
markdown中无法指定图片的尺寸，如果尺寸很重要的话，可以直接使用&lt;img&gt;标签。

## 杂项（Miscellaneous）
### 自动链接（AUTOMATIC LINKS）

```
<http://example.com/>
```

产生：
```
<a href="http://example.com/">http://example.com/</a>
```

效果：<http://example.com/>

```
<address@example.com>
```

产生：
```
<a href="&#x6D;&#x61;i&#x6C;&#x74;&#x6F;:&#x61;&#x64;&#x64;&#x72;&#x65;
&#115;&#115;&#64;&#101;&#120;&#x61;&#109;&#x70;&#x6C;e&#x2E;&#99;&#111;
&#109;">&#x61;&#x64;&#x64;&#x72;&#x65;&#115;&#115;&#64;&#101;&#120;&#x61;
&#109;&#x70;&#x6C;e&#x2E;&#99;&#111;&#109;</a>
```

效果：

<address@example.com>

### 反斜线（BACKSLASH ESCAPES）

markdwon可以用反斜杠来生成一些字面量：

```
\   backslash
`   backtick
*   asterisk
_   underscore
{}  curly braces
[]  square brackets
()  parentheses
#   hash mark
+   plus sign
-   minus sign (hyphen)
.   dot
!   exclamation mark
```

效果：

\\
\`
\*
\_
\{\}
\[\]
\(\)
\#
\+
\-
\.
\!

### 锚点
（1）标题默认做锚点。
使用 
```
## HeaderName
```
生成的标题都会用一个`id="HeaderName"`的元素包裹。同一页面可以用这样引用：`[link text](#HeaderName)`，其他页面可以这样引用：`[link text](http://...#HeaderName)`。
也可以直接使用`[Header Name][]`引用这个标题。

（2）使用`<a name="abcd"></a>`创建锚点：
```
#### <a name="header1234"></a>A Heading in this SO entry!
```
produces:
#### <a name="header1234"></a>A Heading in this SO entry!
然后可以通过下面的方式链接到这里：

```
 Click [Here](#header1234) to link 
```
Click [Here](#header1234) to link 
  也可以使用`<a id="abcd"></a>`创建锚点，但不推荐，原因看[这里](http://stackoverflow.com/a/7335259/57171)


[链接到标题锚点](#MultiMarkdownOverview)

# GitHub Flavored Markdown
[GitHub Flavored Markdown](https://help.github.com/articles/github-flavored-markdown/)(缩写为GFM)是标准markdwon的一个扩展集，在github上面大放异彩。

## 与传统Markdwon的区别
### 文字中的下划线
Markdown会把下划线(`_`)包裹的文本转换成斜体，但GFM会忽略文字中的下划线，比如：
```
wow_great_stuff
do_this_and_do_that_and_another_thing.
```
如果需要强调时，请用星号(`*`)

### URL自动链接
GFM对标准的URLs会自动生成链接，比如：
```
http://example.com
```
会变成：
http://example.com

### 中划线（Strikethrough)
GFM添加了中划线的语法：
```
~~Mistaken text.~~
```
变成：
~~Mistaken text.~~
### 包裹的代码块（Fenced code blocks）
标准Markdown中用空行，4个空格（或tab）缩进来代表代码块，GFM使用三个反引号包裹代码块(\`\`\`)

### 语法高亮（Syntax highlighting）
下面代码是一段ruby的代码：

```ruby
    require 'redcarpet'
    markdown = Redcarpet.new("Hello World!")
    puts markdown.to_html   
```

### 表格（Tables）
用破折号隔离表头和表体，用pipe(|) 隔离每一列：
```
First Header  | Second Header
------------- | -------------
Content Cell  | Content Cell
Content Cell  | Content Cell
```
效果：（要加空行）

First Header  | Second Header
------------- | -------------
Content Cell  | Content Cell
Content Cell  | Content Cell

也可以更加美观：
```
| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |
```

注意，破折号的长度不需要匹配表头长度：
```
| Name | Description          |
| ------------- | ----------- |
| Help      | Display the help window.|
| Close     | Closes a window     |
```
可以在单元格里面使用内联Markdown标签：链接、粗体、斜体、中划线，等：

| Name | Description          |
| ------------- | ----------- |
| [Help](http://example.com)      | ~~Display the~~ help window.|
| Close     | _Closes_ a window     |

可以用冒号（:）来定义列的对其方式：
```
| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |
```
效果如下：

| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |

### HTML
可以使用一个HTML的子集，所有支持的标签和属性可以在[github/markup repository](https://github.com/github/markup)找到。

## 扩展阅读
["Writing on GitHub"](https://help.github.com/articles/writing-on-github/)

# Markdown编辑器推荐
目前为止，我使用Markdown体验最好的是:[Sublime Text 3](http://www.sublimetext.com/3) + **Markdown Editing**,[这里有篇文章介绍](http://www.cnblogs.com/IPrograming/p/Sublime-markdown-editor.html)，基本的Markdown标签都是所见即所得的：
![示例图片](http://images.cnitblog.com/blog2015/294941/201504/251326002036398.png)

另外，我也试过Webstorm和eclipse的插件，感觉没那么好用，最大的区别就是，这些工具都有两个窗口：一个源码Editor窗口，一个Preview窗口。

webstorm
http://blog.fens.me/webstorm-markdown/

eclipse plugin 编辑和查看：
https://github.com/winterstein/Eclipse-Markdown-Editor-Plugin
http://marketplace.eclipse.org/content/markdown-text-editor
http://marketplace.eclipse.org/content/github-flavored-markdown-viewer-plugin-eclipse

**当然sublime text 3 + Markdown Editing 配合 hexo 是目前我采用的方式。**
 
