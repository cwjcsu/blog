title: 轻松管理100台云主机不是梦-云管家试用心得
date: 2016-08-26 15:13:42
tags:
- 阿里云
- 腾讯云
- 云管家
categories:
- 工具
description: 轻松管理100台云服务器不是梦-行云服务-云管家试用心得
---
自从公司买了几十台阿里云ECS服务器以来，每天设置告警、统计费用成本、整理IP地址和登录密钥。。。那么ugly的复杂无比的控制台。。。每天上百条不知所云的告警短信和邮件。。。搞得我崩溃了，被老板吊了好几次。直到我试用行云服务的[云管家](https://yun.cloudbility.com/)以后———世界终于安静了！腰不酸腿不疼了，有精力yy性生活了~~

<!--more-->

### 概述
为了满足大家的好奇心，我先概述一下云管家的以下特性：
1. 只需要三步（注册登录，创建团队，用access key导入阿里云主机），然后就看到了所有主机的概览和状态。
2. 在网页上以ssh-client方式访问所有主机（用key或者密码），试用了一下这个网页版的ssh工具，比阿里云原生那个要好。**访问过程竟然还有录像。**
3. 默认就有 **微信告警通知。** （我是用微信注册的，注册的时候关注了云管家的公众号。注册以后在个人资料里面绑定微信也可以）
4. 所有主机以及成本明细一目了然，云管家对主机负载进行了分析，给出了 **优化建议**。比如建议我把一台访问量比较低的主机的带宽设置为按量计费。
5. 体检中心扫出有几台机器开放了危险端口，还有 **防火墙没打开** 也提醒我了。

### 幸好没被盯上

前段时间，为了图方便，我把mysql数据库端口3306暴率出来（当时比较急，需要运程调试一下生成环境，然后忘记了，你懂的），当看到云管家提醒我暴露危险端口时，感觉菊花一紧，幸好没被ddos攻击盯上，否则。。。我脑海中浮现出老板气急败坏的那张脸。

后来，我把所有主机做了一次安全加固，ssh禁用密码访问，开启防火墙，只暴露几个http和https的端口，等等。

### 查看更多
总的来说，这个工具做的确实比较用心，现在还处于beta阶段，希望可以做的更好更稳定。
闲话不说，[可以去这里直接把玩一下](https://yun.cloudbility.com/)，这里还有一篇[试用帖子](http://jeremyxu2010.github.io/2016/08/25/%E8%AF%95%E7%94%A8%E4%BA%91%E7%AE%A1%E5%AE%B6%E7%B3%BB%E5%88%9701/)，描述的比较全面。







