title: MathJax与LaTeX教程与速查手册 - MathJax and LaTeX basic tutorial and quick reference
date: 2015-10-21 20:42:22
tags:
- MathJax
- LaTeX

categories:
- 工具
 
description: MathJax与LaTeX教程与速查手册 MathJax and LaTeX basic tutorial and quick reference
---
MathJax与LaTex教程与速查手册。以及LaTeX在Markdown中使用的注意事项。

<!--more-->
## 速查手册
### 查看TeX命令
 查看本页的公式是如何编写，右击表达式，选择 "Show Math As > Tex Commands"就可以显示了。
### 内联和公式块
1. 内联公式，用美元符号`$...$`包裹，比如`$\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}$`产生:$\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}$.
1. 换行的块，用两对美元符号`$$...$$`包裹，如`$$\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}\tag{displayed}$$`产生

$$\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}\tag{displayed}$$

### 希腊字母
对于**希腊字母**,用`\alpha`, `\beta`, …, `\omega`: $\alpha,\beta,…,\omega$. 大写的，可以使用`\Gamma`, `\Delta`, …, `\Omega`:$ \Gamma,\Delta,…,\Omega$等等。

### 上角下角符号
对于**上角文字和下角文字**，分别使用`^`和`_`，比如：`x_i^2`:$x_i^2$

用花括号`{...}`包裹一个原子的符号或者表达式，比如用`10^10`会得到:$10^10$，正确的写法是：`10^{10}`:$10^{10}$。另外，这种写法：`x^5^6`是错误的，需要用花括号进行界定。`{x^y}^z`是:${x^y}^z$，`x^{y^z}`是：$x^{y^z}$。注意`x_i^2`:$x_i^2$和`x_{i^2}`:$x_{i^2}$。

### 花括号
1. 圆括号和方括号`()[]`用法就是普通的用法，比如`(2+2)[4-4]`:$(2+2)[4-4]$。
   而表达式中的花括号需要用`\\{`和`\\}`表示，比如`\{x| x>0\}`:$\\{x| x>0\\}$

1. 默认插入会被压缩变小：`(\frac{\sqrt x}{y^3})`:$(\frac{\sqrt x}{y^3})$,使用`\left(…\right)`可以自动调整行高,`\left(\frac{\sqrt x}{y^3}\right)` :$\left(\frac{\sqrt x}{y^3}\right)$。
1. `\left` 和 `\right` 适用于下面的各种符号：`(`和`)`,$\left((x)\right)$； `[` 和 `]`:\left([x]\right)；`\{` and `\}`:$\left(\\{x\\}\right)$； `|`：$\left(|x|\right)$； `\langle` and `\rangle`：$\left(\langle x \rangle\right)$；,  `\lceil` and `\rceil`:$\left(\lceil x \rceil\right)$, and `\lfloor` and `\rfloor`：$\left(\lfloor x \rfloor\right)$；也可以用`.`进行隐式插入：`\left.\frac12\right\rbrace` : $\left.\frac12\right\rbrace$.

### 求和与积分
求和与积分:`\sum_1^n`,`\int` :$\sum_1^n$,`\sum_{i=0}^\infty i^2` :$\sum_{i=0}^\infty i^2$;乘积：`\prod`:$\prod$;`\int` :$\int$;`\bigcup`:$\bigcup$, `\bigcap`: $\bigcap$, `\iint`:$\iint$.

### 分数
分数有两种方式。
1. `\frac ab`:$\frac ab$,`\frac {a+1}{b+2}`:$\frac {a+1}{b+2}$;
1. `\over`，`{a+1\over b+1}`:${a+1\over b+1}$.

### 字体：
* Use \mathbb or \Bbb for "blackboard bold": $\mathbb{CHNQRZ}$;
* Use \mathbf for boldface: $mathbf{ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz}$;
* Use \mathrm for roman font:$\mathrm{ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz}$;
* Use \mathsf for sans-serif font:$\mathsf{ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz}$;
* Use \mathcal for "calligraphic" letters:$\mathcal{ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz}$;
* Use \mathscr for script letters:$\mathscr{ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz}$;
* Use \mathfrak for "Fraktur" (old German style) letters:$\mathfrak{ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz}$;

### 开根号
使用`\sqrt` :$\sqrt{x^3}$,`\sqrt[3]{\frac xy}`:$\sqrt[3]{\frac xy}$,对于复杂表达式，可以使用：`{...}^{1/2}`.

### 特殊函数
比如："lim", "sin", "max", "ln"，使用`\lim`,`\sin`等等。`\sin x`:$\sin x$;
极限：`\lim_{x\to 0}`:$$\lim_{x\to 0}$$.

### 特殊符号
有很多，有个[简表](http://pic.plover.com/MISC/symbols.pdf)和[全表](http://mirror.math.ku.edu/tex-archive/info/symbols/comprehensive/symbols-a4.pdf)可以查询。最常用的包括：
* `\lt \gt \le \ge \neq` :$\lt \gt \le \ge \neq$,`\not\lt` :$\not\lt$;
* `\times \div \pm \mp` :$\times \div \pm \mp$，中间点是`\cdot`:$x\cdot y$;
* `\cup \cap \setminus \subset \subseteq \subsetneq \supset \in \notin \emptyset \varnothing`:$\cup \cap \setminus \subset \subseteq \subsetneq \supset \in \notin \emptyset \varnothing$;
* `{n+1 \choose 2k}` or `\binom{n+1}{2k}` :$\binom{n+1}{2k}$;
* `\to \rightarrow \leftarrow \Rightarrow \Leftarrow \mapsto` :$\to \rightarrow \leftarrow \Rightarrow \Leftarrow \mapsto$;
* `\land \lor \lnot \forall \exists \top \bot \vdash \vDash`:$\land \lor \lnot \forall \exists \top \bot \vdash \vDash$;
* `\star \ast \oplus \circ \bullet` :$\star \ast \oplus \circ \bullet$;
* `\approx \sim \simeq \cong \equiv \prec` :$\approx \sim \simeq \cong \equiv \prec$;
* `\infty \aleph_0`: $\infty \aleph_0$;`\nabla \partial`:$\nabla \partial$;`\Im \Re IR`:$\Im \Re IR$;
* `\pmod` : `a\equiv b\pmod n`:$a\equiv b\pmod n$;
* `\ldots`:$a_1,a_2,\ldots,a_n$, `\cdots` :$a_1+a_2+\cdots+a_n$;
* 一些希腊字母的变种：`\epsilon \varepsilon`:$\epsilon \varepsilon$, `\phi \varphi`:$\phi \varphi$, and others. Script lowercase l is `\ell`:$\ell$.
[这里可以查看更多](http://detexify.kirelabs.org/classify.html)
[MathJax支持的LaTex命令](http://docs.mathjax.org/en/latest/tex.html#supported-latex-commands)
[MathJax常用命令](http://www.onemathematicalcat.org/MathJaxDocumentation/TeXSyntax.htm)

### 空格
常量空格MathJax并不会起作用，而要用`\,`,比如:$a\,b$.对于很宽的空格使用`\quad` 和 `\qquad`:$a \quad b,a \qquad b$.

### 纯文本
`\text{…}`:$\\{x\in s\mid x\text{ is extra large}\\}$

### 字母上面的符号
* `\hat`,`\hat x` :$\hat x$;
* `\widehat`给多个字母`\widehat{xy}`:$\widehat{xy}$;
* `\bar`,`\bar x` :$\bar x$;
* `\overline`,`\overline {xyz}` :$\overline {xyz}$;
* 向量`\vec`,`\vec x` :$\vec x$;
* `\overrightarrow`,`\overrightarrow {xy}`:$\overrightarrow {xy}$;
* `\overleftrightarrow`,'\overleftrightarrow {xy}':$\overleftrightarrow{xy}$;
* 点:`\frac d{dx}x\dot x =  \dot x^2 +  x\ddot x` :$\frac d{dx}x\dot x =  \dot x^2 +  x\ddot x$;

### MathJax特殊字符

Special characters used for MathJax interpreting can be escaped using the `\` character: `\\$`:$\\$$, `\\{`:$\\{$, `\\_`:$\\_$, etc. If you want `\` itself, you should use `\backslash`:$\backslash$, because `\\` is for a new line.
(**Note**:发现Hexo生成的markdown里面反斜杠数目要加倍，比如，MathJax用`\{`表示左花括号，在markdown里面要用`\\{`，而markdown里面换行要用:`\\\\`)

## 其他示例
### 矩阵
```
$$
        \begin{matrix}
        1 & x & x^2 \\\\
        1 & y & y^2 \\\\
        1 & z & z^2 \\\\
        \end{matrix}
$$
```
产生：
$$
        \begin{matrix}
        1 & x & x^2 \\\\
        1 & y & y^2 \\\\
        1 & z & z^2 \\\\
        \end{matrix}
$$



### 分情况定义
```
 f(n) =
\begin{cases}
n/2,  & \text{if $n$ is even} \\
3n+1, & \text{if $n$ is odd}
\end{cases}
```
产生
 $$f(n) =
\begin{cases}
n/2,  & \text{if $n$ is even} \\\\
3n+1, & \text{if $n$ is odd}
\end{cases}$$

### 数组
```
$$
\begin{array}{c|lcr}
n & \text{Left} & \text{Center} & \text{Right} \\
\hline
1 & 0.24 & 1 & 125 \\
2 & -1 & 189 & -8 \\
3 & -20 & 2000 & 1+10i
\end{array}
$$
```
产生：
$$
\begin{array}{c|lcr}
n & \text{Left} & \text{Center} & \text{Right} \\\\
\hline
1 & 0.24 & 1 & 125 \\\\
2 & -1 & 189 & -8 \\\\
3 & -20 & 2000 & 1+10i
\end{array}
$$

### 方程
```
$$
\left\{ 
\begin{array}{c}
a_1x+b_1y+c_1z=d_1 \\ 
a_2x+b_2y+c_2z=d_2 \\ 
a_3x+b_3y+c_3z=d_3
\end{array}
\right. 
$$
```
产生：
$$
\left\\{ 
\begin{array}{c}
a_1x+b_1y+c_1z=d_1 \\\\ 
a_2x+b_2y+c_2z=d_2 \\\\
a_3x+b_3y+c_3z=d_3
\end{array}
\right. 
$$


## 参考文档
这里面有更多：
[http://meta.math.stackexchange.com/ 上面一个完成的教程](http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference)
[mathjax示例](http://cdn.mathjax.org/mathjax/latest/test/examples.html)



