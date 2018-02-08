title: Socks协议详解以及java SocksSocketImpl处理域名的bug
date: 2018-02-08 11:16:30
tags: 
- java
categories:
- 编程
description: JDK一个bug:SocksSocketImpl处理Socks代理时，错误的获取域名地址的长度导致建立连接失败。
---
为了描述这个bug，需要把socks的协议过一遍，socks协议的rfc文档可以看[这里](https://tools.ietf.org/html/rfc1928)。socks协议是建立在TCP连接上的，socks proxy连接也是一个普通的TCP socket连接，只是在socket处理具体工作之前，会处理代理相关的一些指令。
<!--more-->
# 授权方法协商
建立socket之后，客户端发送一个版本信息码：
```
                   +----+----------+----------+
                   |VER | NMETHODS | METHODS  |
                   +----+----------+----------+
                   | 1  |    1     | 1 to 255 |
                   +----+----------+----------+
```
查看SocksSocketImpl.java的`void connect(SocketAddress endpoint, int timeout)`方法：(以socks v5举例)
```
        out.write(PROTO_VERS);//4 or 5
        out.write(2);// 授权方法的种类，2种：不授权和用户密码授权
        out.write(NO_AUTH);
        out.write(USER_PASSW);
```
后面在`METHODS`字段列出两个授权方法的代码：`NO_AUTH`和`USER_PASSW`，`METHODS`字段是变长的，可以是1到255个，这里是2个。

服务器端选择客户端支持的一种授权方法，发送给客户端：
```
                         +----+--------+
                         |VER | METHOD |
                         +----+--------+
                         | 1  |   1    |
                         +----+--------+
```

`METHOD`字段支持的授权方法列表定义为：
* X'00' NO AUTHENTICATION REQUIRED
* X'01' GSSAPI
* X'02' USERNAME/PASSWORD
* X'03' to X'7F' IANA ASSIGNED
* X'80' to X'FE' RESERVED FOR PRIVATE METHODS
* X'FF' NO ACCEPTABLE METHODS
客户端获取到授权方法之后，就开始特定方法的协商。

# Socks代理请求
客户端发送的数据格式为：
```
        +----+-----+-------+------+----------+----------+
        |VER | CMD |  RSV  | ATYP | DST.ADDR | DST.PORT |
        +----+-----+-------+------+----------+----------+
        | 1  |  1  | X'00' |  1   | Variable |    2     |
        +----+-----+-------+------+----------+----------+
```
其中
* VER    协议版本: X'05'
* CMD  发起的指令：
    * CONNECT X'01'
    * BIND X'02'
    * UDP ASSOCIATE X'03'
* RSV    保留
* ATYP   地址类型，分别是：
    * IP V4 地址: X'01'
    * 域名地址: X'03'
    * IP V6 地址: X'04'
* DST.ADDR  变长变量，连接目标的域名或者ip地址
* DST.PORT 两个字节代表连接目标的端口

# 对连接目标寻址
`DST.ADDR`是客户端要实际连接的目标地址，而不是Socks服务的地址，`DST.ADDR`内容这么分，当ATYP的取值为：
* X'01'，即IPv4时，是4个字节的ipv4地址
* X'03'，即域名时，第一个前导byte是域名的长度信息，后面是域名的字符信息
* X'04'，即IPv6时，是16个字节的ipv6地址
从协议上可以看出，socks代理不支持访问域名长度超过255的地址。
看JDK代码:
```
if (epoint.isUnresolved()) {
            out.write(DOMAIN_NAME);
            out.write(epoint.getHostName().length());
            try {
                out.write(epoint.getHostName().getBytes("ISO-8859-1"));
            } catch (java.io.UnsupportedEncodingException uee) {
                assert false;
            }
            out.write((epoint.getPort() >> 8) & 0xff);
            out.write((epoint.getPort() >> 0) & 0xff);
        } else if (epoint.getAddress() instanceof Inet6Address) {
            out.write(IPV6);
            out.write(epoint.getAddress().getAddress());
            out.write((epoint.getPort() >> 8) & 0xff);
            out.write((epoint.getPort() >> 0) & 0xff);
        } else {
            out.write(IPV4);
            out.write(epoint.getAddress().getAddress());
            out.write((epoint.getPort() >> 8) & 0xff);
            out.write((epoint.getPort() >> 0) & 0xff);
}
```
<a name="useDomain">同时，前两篇博客说到的，不在客户端而是在Proxy服务器端解析域名的原理就是来自这里。：
```
if (epoint.isUnresolved()) {
            out.write(DOMAIN_NAME);
            ...
}
```
如果传入的InetSocketAddress是unresolved，那么ATYP采用域名的模式，否则会直接使用解析的IP地址，即采用IPv4或者IPv6。

# 请求的响应
```
  +----+-----+-------+------+----------+----------+
        |VER | REP |  RSV  | ATYP | BND.ADDR | BND.PORT |
        +----+-----+-------+------+----------+----------+
        | 1  |  1  | X'00' |  1   | Variable |    2     |
        +----+-----+-------+------+----------+----------+

     Where:

          o  VER    protocol version: X'05'
          o  REP    Reply field:
             o  X'00' succeeded
             o  X'01' general SOCKS server failure
             o  X'02' connection not allowed by ruleset
             o  X'03' Network unreachable
             o  X'04' Host unreachable
             o  X'05' Connection refused
             o  X'06' TTL expired
             o  X'07' Command not supported
             o  X'08' Address type not supported
             o  X'09' to X'FF' unassigned
          o  RSV    RESERVED
          o  ATYP   address type of following address
             o  IP V4 address: X'01'
             o  DOMAINNAME: X'03'
             o  IP V6 address: X'04'
          o  BND.ADDR       server bound address
          o  BND.PORT       server bound port in network octet order
```
为了解释清楚BND.ADDR的含义，有必要画一张图：socks代理的过程是这样的：
```
Client <--> SocksServer <--> Target
```

Client不能直接与Target建立连接，Client先连接到SocksServer，再由SocksServer连接Target。
在这里，(BND.ADDR,BND.PORT)是SocksServer与Target建立连接后本地绑定的地址，即包的出发地址，不是Target上的目标地址，所以，BND.ADDR通常是SocksServer的IP或者域名。
看JDK代码：，先读取前四个字节：`VER,REP,RSV,ATYP`
```
 data = new byte[4];
 readSocksReply(in, data, deadlineMillis);
  switch (data[1]) {//that's REP
        case REQUEST_OK:
            // success!
            switch(data[3]) {
            case IPV4:
                addr = new byte[4];
                i = readSocksReply(in, addr, deadlineMillis);
                if (i != 4)
                    throw new SocketException("Reply from SOCKS server badly formatted");
                data = new byte[2];
                i = readSocksReply(in, data, deadlineMillis);
                if (i != 2)
                    throw new SocketException("Reply from SOCKS server badly formatted");
                break;
            case DOMAIN_NAME:
                len = data[1];
                byte[] host = new byte[len];
                i = readSocksReply(in, host, deadlineMillis);
                if (i != len)
                    throw new SocketException("Reply from SOCKS server badly formatted");
                data = new byte[2];
                i = readSocksReply(in, data, deadlineMillis);
                if (i != 2)
                    throw new SocketException("Reply from SOCKS server badly formatted");
                break;
            case IPV6:
                len = data[1];
                addr = new byte[len];
                i = readSocksReply(in, addr, deadlineMillis);
                if (i != len)
                    throw new SocketException("Reply from SOCKS server badly formatted");
                data = new byte[2];
                i = readSocksReply(in, data, deadlineMillis);
                if (i != 2)
                    throw new SocketException("Reply from SOCKS server badly formatted");
                break;
            default:
                ex = new SocketException("Reply from SOCKS server contains wrong code");
                break;
            }
            break;
        case GENERAL_FAILURE:
            ex = new SocketException("SOCKS server general failure");
            break;
        ...
        }
    }
```

注意到这几行代码：
```
switch(data[3]) {
      case DOMAIN_NAME:
                len = data[1];
                ...
            }
}
```
如果返回的BND.ADDR是域名，则先读取域名的长度，长度取的值是：`REP`，也就是0。后面会出一些列异常。

# Netty example错误理解·`BND.ADDR`让我发现了这个bug
这么多年了，一直没人发现这个bug，我想原因大概是，SocksServer返回这个`BND.ADDR`基本都是使用IPv4吧，一个本地socket端口用域名返回的情况几乎不存在。
在`io.netty.example.socksproxy.SocksServerConnectHandler#channelRead0`里面有代码如下：
```
  final Socks5CommandRequest request = (Socks5CommandRequest) message;
            Promise<Channel> promise = ctx.executor().newPromise();
            promise.addListener(
                    new FutureListener<Channel>() {
                        @Override
                        public void operationComplete(final Future<Channel> future) throws Exception {
                            final Channel outboundChannel = future.getNow();
                            if (future.isSuccess()) {
                                ChannelFuture responseFuture =
                                        ctx.channel().writeAndFlush(new DefaultSocks5CommandResponse(
                                                Socks5CommandStatus.SUCCESS,
                                                request.dstAddrType(),//【1】
                                                request.dstAddr(),//【2】
                                                request.dstPort()));//【3】

                                responseFuture.addListener(new ChannelFutureListener() {
                                    @Override
                                    public void operationComplete(ChannelFuture channelFuture) {
                                        ctx.pipeline().remove(SocksServerConnectHandler.this);
                                        outboundChannel.pipeline().addLast(new RelayHandler(ctx.channel()));
                                        ctx.pipeline().addLast(new RelayHandler(outboundChannel));
                                    }
                                });
                            } else {
                                ctx.channel().writeAndFlush(new DefaultSocks5CommandResponse(
                                        Socks5CommandStatus.FAILURE, request.dstAddrType()));
                                SocksServerUtils.closeOnFlush(ctx.channel());
                            }
                        }
                    });
```

意思是连接Target成功后发送一个响应包`DefaultSocks5CommandResponse`，【1】【2】【3】这三处代码使用了请求Target的地址类型、地址、端口作为`ATYP，BND.ADDR， BND.PORT`，实际上应该用SocksServer本机的IP和端口。

所以，当我使用unresolve的地址请求代理服务器的时候，就重现了这个bug。
正确的写法是这样的：
```
if (future.isSuccess()) {
   InetSocketAddress socketAddress = (InetSocketAddress) outboundChannel.localAddress();
                                //README 返回addrType必须为ipv4，因为jre SocksSocketImpl有bug，无法正确处理响应DOMAIN_NAME和IPV6的响应
                                ChannelFuture responseFuture =
                                        ctx.channel().writeAndFlush(new DefaultSocks5CommandResponse(
                                                Socks5CommandStatus.SUCCESS,
                                                Socks5AddressType.IPv4,
                                                socketAddress.getAddress().getHostAddress(),
                                                socketAddress.getPort()));
}
```


* [备注1 SOCKS 5 Server - BND.PORT & BND.ADDR](https://stackoverflow.com/questions/43013695/socks-5-server-bnd-port-bnd-addr)