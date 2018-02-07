title: 给OkHttp Client添加socks代理
date: 2018-02-07 10:30:55
tags:
- Java
- Socks Proxy
- okhttp
categories:
- 编程
description: 介绍了如何给okhttp添加socks代理支持，socks代理需要支持用户名、密码的验证，支持https访问。
---
Okhttp的使用没有httpClient广泛，网上关于Okhttp设置代理的方法很少，这篇文章完整介绍了需要注意的方方面面。
<!--more-->
[上一篇博客](/编程/how-to-add-socks-proxy-support-for-httpclient/)中介绍了socks代理的入口是创建`java.net.Socket`时传入一个`java.net.Porxy`对象。 OkHttp client通过`OkHttpClient.Builder`创建，可以通过定制`javax.net.ssl.SSLSocketFactory`和`java.net.SocketFactory`来实现socks代理。
# 定制SocketFactory
JDK中默认的是`java.net.DefaultSocketFactory`,它是包可见的，没法扩展，所以只能用`java.net.SocketFactory`扩展，没有什么其他好的招数，这个过程其实有点繁琐。
```
public class ProxySocketFactory extends SocketFactory {

    private ProxyConfigProvider proxyConfigProvider;

    public ProxySocketFactory(ProxyConfigProvider proxyConfigProvider) {
        this.proxyConfigProvider = proxyConfigProvider;
    }

    @Override
    public Socket createSocket() throws IOException {
        ProxyConfig proxyConfig = proxyConfigProvider.getProxyConfig();
        if (proxyConfig != null) {
            return new Socket(proxyConfig.getProxy());
        } else {
            return new Socket();
        }
    }

    public Socket createSocket(String host, int port)
            throws IOException, UnknownHostException {
        Socket socket = createSocket();
        try {
            socket.connect(new InetSocketAddress(host, port));
        } catch (IOException e) {
            socket.close();
            throw e;
        }
        return socket;
    }

    public Socket createSocket(InetAddress address, int port)
            throws IOException {
        Socket socket = createSocket();
        try {
            socket.connect(new InetSocketAddress(address, port));
        } catch (IOException e) {
            socket.close();
            throw e;
        }
        return socket;
    }

    public Socket createSocket(String host, int port,
                               InetAddress clientAddress, int clientPort)
            throws IOException, UnknownHostException {
        Socket socket = createSocket();
        try {
            socket.bind(new InetSocketAddress(clientAddress, clientPort));
            socket.connect(new InetSocketAddress(host, port));
        } catch (IOException e) {
            socket.close();
            throw e;
        }
        return socket;
    }

    public Socket createSocket(InetAddress address, int port,
                               InetAddress clientAddress, int clientPort)
            throws IOException {
        Socket socket = createSocket();
        try {
            socket.bind(new InetSocketAddress(clientAddress, clientPort));
            socket.connect(new InetSocketAddress(address, port));
        } catch (IOException e) {
            socket.close();
            throw e;
        }
        return socket;
    }
}

```
上面代码这么长，并不是随便写的，也省不了。我参考了`DefaultSocketFactory`里面创建`Socket`对象的各个构造函数，保证了`ProxySocketFactory`的跟`DefaultSocketFactory`对应方法的行为完全一致，只是在创建socket时用了代理而已。简单一句话：只要传入了IP地址和端口，就会直接发起连接。

对SSL socket工厂的定制同样繁琐。不是简单的继承（继承搞不定），而是采用包装模式，用原来的`SSLSocketFactory`来实现与代理无关的方法。
```
public class ProxySSLSocketFactory extends SSLSocketFactory {
    private ProxyConfigProvider configProvider;
    private SSLSocketFactory socketFactory;

    public ProxySSLSocketFactory(ProxyConfigProvider configProvider, SSLSocketFactory socketFactory) {
        this.configProvider = configProvider;
        this.socketFactory = socketFactory;
    }

    @Override
    public String[] getDefaultCipherSuites() {
        return socketFactory.getDefaultCipherSuites();
    }

    @Override
    public String[] getSupportedCipherSuites() {
        return socketFactory.getSupportedCipherSuites();
    }

    public Socket createSocket()
            throws IOException {
        ProxyConfig proxyConfig = configProvider.getProxyConfig();
        if (proxyConfig != null) {
            return new Socket(proxyConfig.getProxy());
        } else {
            return new Socket();
        }
    }

    public Socket createSocket(String host, int port)
            throws IOException {
        Socket socket = createSocket();
        try {
            return socketFactory.createSocket(socket, host, port, true);
        } catch (IOException e) {
            socket.close();
            throw e;
        }
    }

    public Socket createSocket(Socket s, String host,
                               int port, boolean autoClose)
            throws IOException {
        //TODO 无法代理
        return socketFactory.createSocket(s, host, port, autoClose);
    }

    public Socket createSocket(InetAddress address, int port)
            throws IOException {
        Socket socket = createSocket();
        try {
            return socketFactory.createSocket(socket, address.getHostAddress(), port, true);
        } catch (IOException e) {
            socket.close();
            throw e;
        }
    }

    public Socket createSocket(String host, int port,
                               InetAddress clientAddress, int clientPort)
            throws IOException {
        Socket socket = createSocket();
        try {
            socket.bind(new InetSocketAddress(clientAddress, clientPort));
            return socketFactory.createSocket(socket, host, port, true);
        } catch (IOException e) {
            socket.close();
            throw e;
        }
    }

    public Socket createSocket(InetAddress address, int port,
                               InetAddress clientAddress, int clientPort)
            throws IOException {
        Socket socket = createSocket();
        try {
            socket.bind(new InetSocketAddress(clientAddress, clientPort));
            return socketFactory.createSocket(socket, address.getHostAddress(), port, true);
        } catch (IOException e) {
            socket.close();
            throw e;
        }
    }
}
```

注意，其中有一个方法是无法使用代理的：`Socket createSocket(Socket s, String  host,int port, boolean autoClose)`，因为传入的已经是一个创建好的Socket，所以使用这个方法，要注意你的组件有没有用到这个方法，如果用到了，代理的功能需要在这个方法的调用者那一层去实现。我测试OkHttp client是没有用到这个方法的。

# 创建OkHttpClient对象
两个工厂类设计好了之后，下面就是运用他们创建OkHttpClient对象。<a name="code1"></a>
```
    public OkHttpClient buildOkHttpClient() {
        OkHttpClient httpClient = new OkHttpClient.Builder()
                .sslSocketFactory(new ProxySSLSocketFactory(proxyProvider, NOP_TLSV12_SSL_CONTEXT.getSocketFactory()), NOP_TRUST_MANAGER)
                .socketFactory(new ProxySocketFactory(proxyProvider))
                .hostnameVerifier(NOP_HOSTNAME_VERIFIER)
                .connectTimeout(DEFAULT_CONNECTION_TIMEOUT, TimeUnit.SECONDS)
                .writeTimeout(DEFAULT_CONNECTION_TIMEOUT, TimeUnit.SECONDS)
                .readTimeout(DEFAULT_CONNECTION_TIMEOUT, TimeUnit.SECONDS)                
                .addInterceptor(new OkHttpProxyInterceptor())
                .build();
        return httpClient;
    }
```
上面的方法注意两行：
```
.sslSocketFactory(xxx)
.socketFactory(xxx)
```
其他代码，有对超时的设置`.xxxTimeout()`，这里不赘述，还有对https请求证书的设置以及对socks密码的设置，稍后详解

## 设置https需要注意的地方
### `NOP_TRUST_MANAGER`对象设置为信任任何证书：
```
    public static final X509TrustManager NOP_TRUST_MANAGER = new X509TrustManager() {
        @Override
        public void checkClientTrusted(X509Certificate[] x509Certificates, String s)
                throws CertificateException {
        }

        @Override
        public void checkServerTrusted(X509Certificate[] x509Certificates, String s)
                throws CertificateException {
        }

        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[0];
        }
    };
```

### `NOP_HOSTNAME_VERIFIER`对象设置为信任任何域名：
```
    public static final HostnameVerifier NOP_HOSTNAME_VERIFIER = new HostnameVerifier() {
        @Override
        public boolean verify(String s, SSLSession sslSession) {
            return true;
        }
    };
```

### https的协议版本可能不支持TLSv1
上面两条设置网上很多文档都会介绍，重点需要关注的是`NOP_TLSV12_SSL_CONTEXT`对象。

干货来了：JDK使用的HTTPS协议版本参考[这里-diagnosing-tls,-ssl,-and-https](https://blogs.oracle.com/java-platform-group/diagnosing-tls,-ssl,-and-https)，可以看到在JDK7以前，默认都是TLSv1，JDK8才默认采用TLSv1.2，而很多较新的网站都已经禁用HTTPS使用TLSv1握手了，认为这个协议已经不够安全（例如，我在给minio-java client添加proxy支持时，发现minio-server就这么干的）。所以你用java去发起HTTPS连接经常会出现SSL相关的异常，大部分异常信息根本看不懂。

所以，需要在代码里面指定https使用TLSv1.2：
```
    static {
        try {
            NOP_TLSV12_SSL_CONTEXT = SSLContext.getInstance("TLSv1.2");
            NOP_TLSV12_SSL_CONTEXT.init(null, new TrustManager[]{NOP_TRUST_MANAGER}, new java.security.SecureRandom());
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (KeyManagementException e) {
            e.printStackTrace();
        }
    }
```

实际上也可以通过：
```
System.setProperty("https.protocols","TLSv1.2");//在创建任何SSLSocketFactory之前调用
```
或者设置虚拟机参数：
```
-Dhttps.protocols=TLSv1.2,TLSv1.1,TLSv1
```
放在前面的协议会被优先选用

### 检测https支持的协议版本
如果出现SSL相关异常，但是又不确定是不是协议版本导致的，可以有几个工具进行检验：
**nmap** : `nmap --script ssl-enum-ciphers -p 443 baidu.com`,在我机器上看到：
```
PORT    STATE SERVICE
443/tcp open  https
| ssl-enum-ciphers: 
|   SSLv3: 
|     ciphers: 
|       TLS_ECDHE_RSA_WITH_RC4_128_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_RC4_128_SHA (rsa 2048) - A
|     compressors: 
|       NULL
|     cipher preference: server
|     warnings: 
|       CBC-mode cipher in SSLv3 (CVE-2014-3566)
|   TLSv1.0: 
|     ciphers: 
|       TLS_ECDHE_RSA_WITH_RC4_128_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_RC4_128_SHA (rsa 2048) - A
|     compressors: 
|       NULL
|     cipher preference: server
|   TLSv1.1: 
|     ciphers: 
|       TLS_ECDHE_RSA_WITH_RC4_128_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_RC4_128_SHA (rsa 2048) - A
|     compressors: 
|       NULL
|     cipher preference: server
|     warnings: 
|       Weak cipher RC4 in TLSv1.1 or newer not needed for BEAST mitigation
|   TLSv1.2: 
|     ciphers: 
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_RC4_128_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 (secp256r1) - A
|       TLS_RSA_WITH_AES_128_GCM_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_GCM_SHA384 (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_RC4_128_SHA (rsa 2048) - A
|     compressors: 
|       NULL
|     cipher preference: server
|     warnings: 
|       Weak cipher RC4 in TLSv1.1 or newer not needed for BEAST mitigation
|_  least strength: A
```
# 设置Socks的用户名和密码
OkHttp提供了拦截器的机制`okhttp3.Interceptor`，可以定制http发起的流程。
设置密码有两种方式：一种是设置全局的，另一种是按线程隔离（也是就是按请求隔离），不同请求可以设置不同的用户名和密码，详细介绍以及部分代码见[我的上一篇博客-给HttpClient添加Socks代理](/编程/how-to-add-socks-proxy-support-for-httpclient/)。
注意到上面[代码](#code1)有这么一行：
```
.addInterceptor(new OkHttpProxyInterceptor())
```
就是设置一个拦截器。

```
    private class OkHttpProxyInterceptor implements Interceptor {
        @Override
        public Response intercept(Chain chain) throws IOException {
            ProxyConfig proxyConfig = ProxyHttpClientBuilder.this.getProxyConfig();
            boolean clearCredentials = false;
            if (proxyConfig != null) {
                if (proxyConfig.getAuthentication() != null) {
                    ThreadLocalProxyAuthenticator.setCredentials(proxyConfig.getAuthentication());
                    clearCredentials = true;
                }
            }
            try {
                return chain.proceed(chain.request());
            } finally {
                if (clearCredentials) {
                    ThreadLocalProxyAuthenticator.clearCredentials();
                }
            }
        }
    }
```
`ThreadLocalProxyAuthenticator`,`ProxyConfig`等类见上一篇博客。

