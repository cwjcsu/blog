title: 三分法求极值-Ternary Search 
date: 2015-10-21 10:57:53
tags:
- 搜索算法
- 三分法
- 分治算法

categories: 
- 算法

description: 本文介绍了三分法（Ternary Search）的适用场景以及与二分法的区别
---
本文介绍了三分法（Ternary Search）的适用场景，并用一道2015百度笔试题讲解三分法，最后分析了三分法和二分法的区别。
<!--more-->

## 什么是三分法
三分法是求单峰函数的最大值或最小值的算法，是一种[分治算法](https://en.wikipedia.org/wiki/Divide_and_conquer_algorithm)。维基百科：[Ternary_search](https://en.wikipedia.org/wiki/Ternary_search)。

所谓单峰函数，就是凸函数（包括上凸和下凸，严格定义不赘述）。举个例子，假设求函数f(x)的最大值，而且最大值落在区间[A,B]上，为了使用三分法，那么必须存在一个x满足下面的条件：
* 对任意a,b，如果 A ≤ a < b ≤ x ,则f(a) < f(b)，并且
* 对任意a,b，如果 x ≤ a < b ≤ B，则f(a) > F(b)
这样，f(x)就是函数f的最大值。 

## 算法描述
假设函数f(x)在区间[A,B]上有最大值，那么随机取两个点m1和m2 : A < m1 < m2 < B,然后有三种可能：
if f(m1) < f(m2) ,那么最大值不可能在区间[A,m1]上，而是落在(m1,B]上；
if f(m1) > f(m2) ,那么最大值不可能再区间[m2,B]上，而是落在[A,m2)上；
if f(m1) = f(m2) ,那么最大值落在区间[m1,m2]上。

**注意**:上面区间的表示法，用'('和')'表示不包含端点，用'['和']'表示包含端点。

上面每次选取m1,m2可以排除一块区间，即每次划分成三份，根据函数值去掉一份或者两份，正所谓三分法也。
每次可以三等分，即m1和m2的选取可以这样：
* m1 = A + (A-B)/3
* m2 = B - (A-B)/3

时间复杂度：Ｏ(log n)

三分法示意图:
![三分法算法示意图](http://7xn1o8.com1.z0.glb.clouddn.com/ternary_search.png)

## 实例
### 2015百度笔试题（算法）
题目：一个升序数组，可能包含正数负数或者0，求数组中绝对值最小的数，要求复杂度小于O(n)。

题目被发到[伯乐在线 - 2015百度笔试题（算法）](http://group.jobbole.com/9254)。
### 分析
有序数组搜索一个元素，经验主义者会立刻想到二分法，殊不知，本题是求绝对值，数组的绝对值不再是有序的，而是下凸的，即中间小，两边大。所求的正好是最小值。能满足三分法的要求。

### 示例代码
```
    /**
     * 用三分法求升序数组data中绝对值最小的元素。如果有两个绝对值相等，返回值小的那一个。
     */
    public static int findMinAbs(int[] data) {
        int left = 0;
        int right = data.length - 1;
        int m1 = left;
        int m2 = right;
        while (left < right) {
            m1 = left + (right - left) / 3;
            m2 = right - (right - left) / 3;
            int f1 = Math.abs(data[m1]);
            int f2 = Math.abs(data[m2]);
            if (f1 > f2) {//缩小到区间(m1,right]
                left = m1 + 1;
            } else if (f1 < f2) {//缩小到区间[left,m2)
                right = m2 - 1;
            } else { //缩小到区间[m1,m2]
                left = m1;
                right = m2;
                if (left == right - 1) {
                    break;//TODO这里是发现了两个最小值
                }
            }
        }
        return data[left];
    }

    public static void main(String[] args) {
        int[] data = new int[] { -123, -23, -1, 2, 25, 100 };
        System.out.println(findMinAbs(data));
        data = new int[] { -123, -23, -1, 1, 2, 25, 100 };
        System.out.println(findMinAbs(data));
        data = new int[] { 12, 12, 23, 34,112, 125, 1100 };
        System.out.println(findMinAbs(data));
    }
```

输出:
```
-1
-1
12
```


## 与二分法的区别
二分法适用的场景是：从一个有序数组中寻找指定值的元素。二分法适用的是有序数组，三分法适用的是先升后降或者先降后升的数组。