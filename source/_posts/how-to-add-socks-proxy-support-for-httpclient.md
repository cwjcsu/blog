title: 给HttpClient添加Socks代理
date: 2018-02-06 16:21:05
tags:
- Java
- Socks Proxy
- HttpClient
categories:
- 编程
description: 描述如何给http client添加socks代理支持，socks代理需要支持用户名、密码的验证，支持https访问。
---
本文描述http client使用socks代理过程中需要注意的几个方面：1，socks5支持用户密码授权；2，支持https；3，支持让代理服务器解析DNS；
<!--more-->
# 使用代理创建Socket
从原理上来看，不管用什么http客户端（httpclient，okhttp），最终都要转换到`java.net.Socket`的创建上去，看到代码：
```
package java.net;
 public Socket(Proxy proxy) {
    ...
 }
```
这是JDK中对网络请求使用Socks代理的入口方法。（http代理是在http协议层之上的，不在此文讨论范围之内）。
HttpClient要实现socks代理，就需要塞进去一个Proxy对象，也就是定制两个类：`org.apache.http.conn.ssl.SSLConnectionSocketFactory` 和`org.apache.http.conn.socket.PlainConnectionSocketFactory`，分别对应https和http。
代码如下：
```
    private class SocksSSLConnectionSocketFactory extends SSLConnectionSocketFactory {

        public SocksSSLConnectionSocketFactory(SSLContext sslContext, HostnameVerifier hostnameVerifier) {
            super(sslContext, hostnameVerifier);
        }

        @Override
        public Socket createSocket(HttpContext context) throws IOException {
            ProxyConfig proxyConfig = (ProxyConfig) context.getAttribute(ProxyConfigKey);
            if (proxyConfig != null) {//需要代理
                return new Socket(proxyConfig.getProxy());
            } else {
                return super.createSocket(context);
            }
        }

        @Override
        public Socket connectSocket(int connectTimeout, Socket socket, HttpHost host, InetSocketAddress remoteAddress,
                                    InetSocketAddress localAddress, HttpContext context) throws IOException {
            ProxyConfig proxyConfig = (ProxyConfig) context.getAttribute(ProxyConfigKey);
            if (proxyConfig != null) {//make proxy server to resolve host in http url
                remoteAddress = InetSocketAddress
                        .createUnresolved(host.getHostName(), host.getPort());
            }
            return super.connectSocket(connectTimeout, socket, host, remoteAddress, localAddress, context);
        }
    }
```

和
```
    private class SocksSSLConnectionSocketFactory extends SSLConnectionSocketFactory {

        public SocksSSLConnectionSocketFactory(SSLContext sslContext, HostnameVerifier hostnameVerifier) {
            super(sslContext, hostnameVerifier);
        }

        @Override
        public Socket createSocket(HttpContext context) throws IOException {
            ProxyConfig proxyConfig = (ProxyConfig) context.getAttribute(ProxyConfigKey);
            if (proxyConfig != null) {
                return new Socket(proxyConfig.getProxy());
            } else {
                return super.createSocket(context);
            }
        }

        @Override
        public Socket connectSocket(int connectTimeout, Socket socket, HttpHost host, InetSocketAddress remoteAddress,
                                    InetSocketAddress localAddress, HttpContext context) throws IOException {
            ProxyConfig proxyConfig = (ProxyConfig) context.getAttribute(ProxyConfigKey);
            if (proxyConfig != null) {//make proxy server to resolve host in http url
                remoteAddress = InetSocketAddress
                        .createUnresolved(host.getHostName(), host.getPort());
            }
            return super.connectSocket(connectTimeout, socket, host, remoteAddress, localAddress, context);
        }
    }
```
然后在创建httpclient对象时，给HttpClientConnectionManager设置socketFactoryRegistry
```
            Registry<ConnectionSocketFactory> socketFactoryRegistry = RegistryBuilder.<ConnectionSocketFactory>create()
                .register(Protocol.HTTP.toString(), new SocksConnectionSocketFactory())
                .register(Protocol.HTTPS.toString(), new SocksSSLConnectionSocketFactory(sslContext, NoopHostnameVerifier.INSTANCE))
                .build();

        PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager(socketFactoryRegistry);
    
```
# 让代理服务器解析域名
**场景**：运行httpClient的进程所在主机可能并不能上公网，大部分时候，也无法进行DNS解析，这时通常会出现域名无法解析的IO异常，下面介绍怎么避免在客户端解析域名。

上面有一行代码非常关键：
```
remoteAddress = InetSocketAddress 
                        .createUnresolved(host.getHostName(), host.getPort());
```
变量`host`是你发起http请求的目标主机和端口信息，这里创建了一个未解析(Unresolved)的SocketAddress，在socks协议握手阶段，InetSocketAddress信息会原封不动的发送到代理服务器，由代理服务器解析出具体的IP地址。
Socks的[协议描述](https://tools.ietf.org/html/rfc1928)中有个片段：
```
   The SOCKS request is formed as follows:

        +----+-----+-------+------+----------+----------+
        |VER | CMD |  RSV  | ATYP | DST.ADDR | DST.PORT |
        +----+-----+-------+------+----------+----------+
        | 1  |  1  | X'00' |  1   | Variable |    2     |
        +----+-----+-------+------+----------+----------+

     Where:

          o  VER    protocol version: X'05'
          o  CMD
             o  CONNECT X'01'
             o  BIND X'02'
             o  UDP ASSOCIATE X'03'
          o  RSV    RESERVED
          o  ATYP   address type of following address
             o  IP V4 address: X'01'
             o  DOMAINNAME: X'03'
             o  IP V6 address: X'04'
```
代码按上面方法写，协议握手发送的是`ATYP=X'03'`，即采用域名的地址类型。否则，HttpClient会尝试在客户端解析，然后发送`ATYP=X'01'`进行协商。当然，大多数时候HttpClient在解析域名的时候就挂了。

# https中需要注意的问题
在使用httpclient访问https网站的时候，经常会遇到javax.net.ssl包中的异常，例如：
```
Caused by: javax.net.ssl.SSLException: Received fatal alert: internal_error
    at sun.security.ssl.Alerts.getSSLException(Unknown Source) ~[na:1.7.0_80]
    at sun.security.ssl.Alerts.getSSLException(Unknown Source) ~[na:1.7.0_80]
```
一般需要做几个设置：
## 创建不校验证书链的SSLContext
```
        SSLContext sslContext = null;
        try {
            sslContext = new SSLContextBuilder().loadTrustMaterial(null, new TrustStrategy() {
                @Override
                public boolean isTrusted(X509Certificate[] chain, String authType)
                        throws CertificateException {
                    return true;
                }

            }).build();
        } catch (Exception e) {
            throw new com.aliyun.oss.ClientException(e.getMessage());
        }
        ...
        new SocksSSLConnectionSocketFactory(sslContext, NoopHostnameVerifier.INSTANCE)
```
## 创建不校验域名的HostnameVerifier
```
public class NoopHostnameVerifier implements javax.net.ssl.HostnameVerifier {

    public static final NoopHostnameVerifier INSTANCE = new NoopHostnameVerifier();

    @Override
    public boolean verify(final String s, final SSLSession sslSession) {
        return true;
    }
}
```

# 如何使用用户密码授权？
java SDK中给Socks代理授权有点特殊，不是按socket来的，而是在系统层面做的全局配置。比如，可以通过下面代码设置一个全局的Authenticator：
```
Authenticator.setDefault(new MyAuthenticator("userName", "Password"));
...
class MyAuthenticator extends java.net.Authenticator {
    private String user ;
    private String password ;
  
    public MyAuthenticator(String user, String password) {
      this.user = user;
      this.password = password;
    }
  
    protected PasswordAuthentication getPasswordAuthentication() {
      return new PasswordAuthentication(user, password.toCharArray());
    }
  }
```
这种方法很简单，不过有些不方便的地方，如果你的产品中需要连接不同的Proxy服务器，而他们的用户名密码是不一样的，那么这个方法就不适用了。

## 基于ThreadLocal的Authenticator
```
public class ThreadLocalProxyAuthenticator extends Authenticator{
    private ThreadLocal<PasswordAuthentication> credentials = null;
     private static class SingletonHolder {
        private static final ThreadLocalProxyAuthenticator instance = new ThreadLocalProxyAuthenticator();
    }
    public static final ThreadLocalProxyAuthenticator getInstance() {
        return SingletonHolder.instance;
    }
      public void setCredentials(String user, String password) {
        credentials.set(new PasswordAuthentication(user, password.toCharArray()));
    }
    public static void clearCredentials() {
        ThreadLocalProxyAuthenticator authenticator = ThreadLocalProxyAuthenticator.getInstance();
        Authenticator.setDefault(authenticator);
        authenticator.credentials.set(null);
    }
    public PasswordAuthentication getPasswordAuthentication() {
        return credentials.get();
    }
}

```
这个类意味着，授权信息只会保存到当前调用者的线程中，其他线程的调用者无法访问，在创建Socket的线程中设置密钥和清理密钥，就可以做到授权按照Socket连接进行隔离。Java TheadLocal相关知识本文不赘述。

## 按连接隔离的授权
```
 class ProxyHttpClient extends CloseableHttpClient{
    private CloseableHttpClient httpClient;
    public ProxyHttpClient(CloseableHttpClient httpClient){
        this.httpClient=httpClient;
    }
    protected CloseableHttpResponse doExecute(HttpHost target, HttpRequest request, HttpContext context) throws IOException, ClientProtocolException {
            ProxyConfig proxyConfig = //这里获取当前连接的代理配置信息
            boolean clearCredentials = false;
            if (proxyConfig != null) {
                if (context == null) {
                    context = HttpClientContext.create();
                }
                context.setAttribute(ProxyConfigKey, proxyConfig);
                if (proxyConfig.getAuthentication() != null) {
                    ThreadLocalProxyAuthenticator.setCredentials(proxyConfig.getAuthentication());//设置授权信息
                    clearCredentials = true;
                }
            }
            try {
                return httpClient.execute(target, request, context);
            } finally {
                if (clearCredentials) {//清理授权信息
                    ThreadLocalProxyAuthenticator.clearCredentials();
                }
            }
        }
 }
```

另外，线程是可以复用的，因为每次调用完毕后，都清理了授权信息。
这里有个一POJO类`ProxyConfig`，保存的是socks代理的IP端口和用户密码信息。
```
public class ProxyConfig {
    private Proxy proxy;
    private PasswordAuthentication authentication;
}
```

