title: 无穷大——你从没听说过的事情 Infinity - You've Never Heard of
date: 2015-10-19 00:00:20
tags: 
- 集合论
- 基数
- 无穷大
- 一一对应
categories:
- 数学

description: 关于三类无穷大的讨论

---
本文从原理上探讨集合的基数，以及无穷大的分类依据。
<!--more-->

## 关于计数
据考古证据显示，原始非洲部落的人对超过3的数是数不清楚的。就是说如果一个人他有4个儿子，他只会说有很多儿子，而不是4个。

虽然原始人不会计数，但当数目比较多时，他们可以判断一个东西的个数是否比另一个东西多。而这条能被原始人理解使用的简单原理也是理解几类无穷大的关键。

### 原始人怎么比较呢？
假设有一堆珠宝和一堆石头，要比较那一堆数目多，只需要每次从珠宝和石头中各拿出来一个，放到其他地方，一直拿，最终会有三种情况：
1. 珠宝拿完了剩下了石头；
2. 石头拿完了剩下了珠宝；
3. 珠宝和石头同时都没有了。
显然，第一种情况石头比珠宝多，第二种情况珠宝比石头多，第三种情况珠宝石头一样多。

### 集合的基数
集合A和集合B有相同的基数，当且仅当存在从A到B的一一对应。
**什么是一一对应呢？**如果你不知道，那高中数学的函数那一章你不可能及格。假设上面的珠宝集合是A，石头集合是B，
* 每拿掉一块珠宝，必须拿掉且仅拿掉一块石头;
* 拿掉的珠宝和石头不能重新放进去；
* 珠宝拿完以后，石头也恰好拿完。
满足上面三个条件，就可以称A-> B存在一个一一对应 （也称为 一一映射）。

### 可数集合
有限集合以及与自然数集合基数相同的集合都称为**可数集合**，其他的集合均是不可数集合。

可数集合包括第一类基数**无穷大**集合 - 零级无穷大。

## 零级无穷大  - 整数个数
### 什么是无穷大？
∞,infinity.简单的说，就是你比任何一个数都大。其实无穷大也是一个数，但你写不出来，它大到什么程度呢？想象一下地球上所有沙子的数目，无穷大比它要大，再大一点，地球上所有原子的数目，无穷大比它还要大。

既然无穷大比任何数都大，那它和自己相比呢？是什么情况。我们知道，所有整数的个数是无穷大，所有偶数的个数也是无穷大，但整数包含奇数和偶数，那你觉得是整数多还是偶数多？如果你不加思索仅凭直觉，你可能会认为整数比偶数多，毕竟整数还包含奇数。

正确答案是：**整数和偶数一样多。** 为什么？
**比较两个无穷大是否相等，关键看它们之间能否建立一个一一对应。 **
我们学一学非洲原始人吧，把整数和偶数分成两堆。从偶数里面拿掉2，相应的从整数里面拿掉1；从偶数里面拿掉4，相应的从整数里面拿掉2；...；从偶数里面拿掉n，从整数里面拿掉n/2...只要你从偶数里面拿掉任何一个，总是能从整数里面也拿掉一个，就是它除以2；相反，你从整数里面拿掉任意一个数m，对应的总是可以从偶数里面也拿掉一个数，就是2m。这样，所有整数和所有偶数就建立了一个对应一一对应关系，不重复不遗漏，所以这两个集合的基数相同。
### 建立有理数到自然数的一一对应
把所有有理数写成最简分数的形式，根据分子和分母的值把它们排列成二维的阵列，然后从1/1出发沿对角线方向蛇形遍历所有的数。第i个遍历到的数与自然数i对应，正有理数集与正整数集也就有了一一对应的关系。我们也可以把正有理数扩展到全体有理数。
![有理数与自然数一一对应示意图](http://7xn1o8.com1.z0.glb.clouddn.com/200801131.gif)

## 一级无穷大  - 实数个数
问：**一条长度为1的线段上面的点数与整数个数那个大？**
我们试着建立一个整数到点的一一对应：线段上的每一个点都可以表示为点到端点的距离，它是一个小数，比如0.72134355451...假设我们找到了一个一一对应,它是这样的：
![任意假定的整数到实数的一一对应](http://7xn1o8.com1.z0.glb.clouddn.com/infinity1.png)
看到这个映射以后，我们还可以写出无穷多个不在这个表格里面的小数，怎么写呢？让这个小数的第一个小数位不同于表中第一个小数的第一个小数位，让它的第二个小数位不同于表中第二个小数的第二个小数位,...，例如，可以写出下面的小数：
![找出一个不在上述映射表中的实数](http://7xn1o8.com1.z0.glb.clouddn.com/infinity2.png)
这个数无论如何在上表中是找不到的。这样一一对应就建立不起来。所以这条线段上的点数比整数的个数还要多得多。这就是两类不同的无穷大。

那么，长度为1的线段和长度为2的线段，谁的点多呢？这就跟整数与偶数的比较一样，是一样多的。一一对应很简单：x-> 2x。实际上，不管线段多长，它与长度为1的线段可以建立下面的投影关系：
![长度不同的线段建立一一对应](http://7xn1o8.com1.z0.glb.clouddn.com/infinity3.png)

长度为1的线段与边长为1的正方形的点数是一样多的吗？是的：
![边长为1的正方形与长度为1的线段建立一一对应](http://7xn1o8.com1.z0.glb.clouddn.com/infinity4.png)

同样的，立方体内所有的点数和正 方形或线段上的所有点数相等，只要把代表线段上一个点的无穷小数分作三部分：
![边长为1的立方体与长度为1的线段建立一一对应](http://7xn1o8.com1.z0.glb.clouddn.com/infinity5.png)

## 二级无穷大
上面两种，分别称为0级和1级无穷大。那有没有比线段上的点还要多的无穷大呢？有，那就是所有曲线的数目，称为2级无穷大。目前只发现这三类无穷大（意思是可以被描述和表达出来的）。

**有没有比三类无穷大都大的无穷大？**
有，见下文；
**三类无穷的夹层中间有没有其他无穷大？**
不存在。


## 小结
* 零级无穷大：所有整数的数量
* 一级无穷大：所有小数的数量（等于上面提及的线上所有的点数、面上所有的点数、立方体上所有的点数）
* 二级无穷大：在一张纸上随意地画线条，所有可能画出的线条数目（曲线样式的数目）

零级无穷大 < 一级无穷大 < 二级无穷大
### 三类无穷大的关系
可以证明：**任何一个集合的所有子集所形成的集合的大小比原集合大。**

**最大的无穷大是多大呢？**
答案是没有尽头。


事实上，[0,1）上的实数可以和正整数的所有子集的集合建立一一对应。

把这些实数写成二进制。小数点后第n位为1，对应n在子集中，为0对应不在子集中，这样[0,1)上的实数就和正整数的子集有了一一对应，因此实数和正整数集的所有子集的个数一样多。

也可以证明前面所说曲线可以和实数集的所有子集有一一对应关系。
我们把前面说所曲线看成一个集合，他的所有子集的个数又将比这个集合大。这个过程可以一直进行下去，得到越来越大的无穷大。
　　另外还有一个问题，整数的无穷大和实数的无穷大之间存不存在别的无穷大。也就是说，是否存在比整数的无穷大大，而比实数的无穷大小的无穷大。（连续统假设）
在现有集合公理下面，至今无法证明或证伪。

### 参考文档
[无穷大 - 百度百科](http://baike.baidu.com/view/530029.htm)
[无穷 - 维基百科](https://zh.wikipedia.org/zh/%E6%97%A0%E7%A9%B7)
[《从1到无穷大-科学中的事实与臆测》]()







