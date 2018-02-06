title: 我是如何让minio client上传速度提高几十倍的
date: 2018-02-06 10:17:18
tags:
- java
categories:
- 编程

description: -Djava.compile虚拟机参数对性能的重大影响，导致minio client上传文件速度相差达到几十倍

---
[minio java client](https://github.com/minio/minio-java) 使用okhttp作为底层的http实现，在产品包里面局域网上传文件的速度一直只有400~800KB/s，经过一天排查发现是`-Djava.compile=none`禁用了即时编译导致。
<!--more-->

# 发现问题的场景
minio-java的使用架构图是这样的：
```
 [Minio Server]<--nginx <== Proxy(Socks5 Sever) <== Agent(minio-java on top of okhttp)
 and
 [Minio Server]<--nginx <== Browser(minio-js api)
```
行云管家发布的第一个私有云版本（即4.0），由于网络需要完全隔离，行云管家的团队网盘功能，无法使用阿里云的OSS以及腾讯云的COS作为媒介，经过多方考察，我们选用了[MINIO](https://www.minio.io/)作为服务端存储的解决方案。minio-java适配很顺利，毕竟它是兼容亚马逊S3的，而阿里云的OSS跟S3穿的是一条裤子，minio-java底层使用的是okhttp，添加socks代理支持也不在话下。

问题来了，正式发版以后，一个同事测试文件采集，上传一个1.2G的文件用了50分钟左右，私有部署跟Proxy以及Agent都安装在一台机器上，带宽理论上来说至少是百兆，速度应该不会低于10MB/s才正常。实测发现，Agent在minio-java情况下的上传速度不超过800KB/s.

# 排查问题
## 是不是MINIO Server的问题？
检查了MINIO Server的配置，都是是默认的，使用团队网盘网页上传文件速度能接近带宽速度（minio-js api），证明MINIO Server和前置的nginx都没有问题。
## 是不是okhttp使用代理导致的？
Socks5代理Server是在Proxy进程里面启动的，从开发上线到现在只经历大概两个版本的迭代，如果有隐藏bug确实不容易暴露的。我这么排查：还是用Agent采集文件，不过网盘分别是OSS和COS，速度都可以达到2~5MB/s，都走Socks5代理，这个速度是正常的，远远超过minio-java的速度。所以可以证明不是代理导致（我的代理程序还是相当靠谱的）。

## 定位到Wrapper
我在idea里面启动Agent实例测试，发现速度接近带宽速度。idea里面的进程跟安装后的Agent最大的区别是：Agent是被[java service wrapper](https://wrapper.tanukisoftware.com/doc/english/download.jsp)包装启动的，而Java代码是一模一样的。于是我替换Agent的jre，调整内存等参数，都没找到问题。

然后我升级了wrapper的版本到最新版，速度依然很慢。

## 定位到`java.compile=none`
我把速度正常的进程启动参数和安装Agent的启动参数拿下来逐一对比，排除了一些明显不可能的参数之后，发现安装Agent里面有一个参数`java.compile=none`，把这个参数去掉以后，Agent文件采集的速度果然恢复正常：达到15MB/s。

# 速度慢的原因
Java程序是通过解释器（Interpreter）进行解释执行的，当虚拟机发现某些代码执行非常频繁时，会把这些代码认定为**热点代码**。为了提高热点代码的执行效率，在运行时，即时编译器（JIT，Just In Time Compiler）会把热点代码编译成本地平台相关的机器码。随着时间的推移，编译器逐渐发挥作用，把越来越多的代码编译成本地代码之后，可以获得更高的执行效率。

虚拟机参数：`java.compile=none`是禁用即时编译的意思，去掉这个配置即可（默认是开启的）。我已经忘记这个参数是怎么配置上去的了，或许是wrapper的默认配置？
