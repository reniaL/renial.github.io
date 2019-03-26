
---
layout: post
title:  "记一次线程挂死的排查过程（附 HttpClient 配置建议）"
tags: [code]
---

## 1、事发

我们有个视频处理程序，基于 SpringBoot，会启动几个线程来跑。要退出程序时，会发送一个信号给程序，每个线程收到信号后会平滑退出，等全部线程都退出后，整个进程再平滑退出。

整个程序平时运行都正常，然后有一天，我们发送了退出信号给程序后，发现程序无法自动退出了！肿么回事呢，grep 一下日志看到是这样的。

```js
# grep 'receive exit signal' /PATH/TO/LOG

[2019-02-22 09:49:28,884][INFO ][Thread-75][n.p.j.e.l.l.PolyvQueueVideo:83] - receive exit signal ... exit current thread
[2019-02-22 09:49:56,271][INFO ][Thread-78][n.p.j.e.l.l.PolyvQueueVideo:83] - receive exit signal ... exit current thread
[2019-02-22 09:53:24,943][INFO ][Thread-74][n.p.j.e.l.l.PolyvQueueVideo:83] - receive exit signal ... exit current thread
[2019-02-22 09:55:23,317][INFO ][Thread-79][n.p.j.e.l.l.PolyvQueueVideo:83] - receive exit signal ... exit current thread
[2019-02-22 09:57:00,196][INFO ][Thread-77][n.p.j.e.l.l.PolyvQueueVideo:83] - receive exit signal ... exit current thread
```

这里的程序总共启动了6个线程的，但上面看到只有5个线程退出了，还有一个哪儿去了？不肯轻易就义么？好顽强的线程。。。

## 2、排查

有点小幸运的是，从上面这5个线程的名称，我们可以推断出那个顽强的线程的名称，从74到79，中间唯独缺了76！那就是你啦 Thread-76！

再查 Thread-76 的日志，确实是我们想找的那个线程，然后发现原来在好几天前它 **好像就停止了运行** ，不再有日志输出。也没有任何异常信息！Thread-76 就这样悄悄的离开，不带走一片云彩。我对着日志和代码大眼瞪小眼看了半个小时，一筹莫展。

此时我想到了福尔摩斯说过的一句话：

> “当你排除掉各种不可能出现的情况之后，剩下的情况无论多么难以置信，都是真相。” -- 福尔摩斯

冷静下来想一想，Thread-76 这个线程，有可能静悄悄地退出了吗，没留下半点异常日志？从理论上来说，不可能。一个线程，要么顺利地执行直到结束，要么中途出错退出了，如果这样肯定有异常信息，但我们并没看到有异常日志。排除掉 “Thread-76 已经退出了” 这个可能性之后，我有个大胆的想法：这个线程还一直运行着！安安静静地运行着，持续着好几天，没有半点日志输出！

是出现了死锁吗？不确定，但我们可以验证一下这个线程是不是真的还存活着。祭出 jstack，把线程信息 dump 出来，一查，果然见到了 Thread-76！

```js
"Thread-76" #141 prio=5 os_prio=0 tid=0x00007f812d7d9800 nid=0x12848 runnable [0x00007f8227cfa000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
        at java.net.SocketInputStream.read(SocketInputStream.java:170)
        at java.net.SocketInputStream.read(SocketInputStream.java:141)
        at java.io.BufferedInputStream.fill(BufferedInputStream.java:246)
        at java.io.BufferedInputStream.read(BufferedInputStream.java:265)
        - locked <0x00000005e64cad10> (a java.io.BufferedInputStream)
        at org.apache.commons.httpclient.HttpParser.readRawLine(HttpParser.java:77)
        at org.apache.commons.httpclient.HttpParser.readLine(HttpParser.java:105)
        at org.apache.commons.httpclient.HttpConnection.readLine(HttpConnection.java:1115)
        at org.apache.commons.httpclient.HttpMethodBase.readStatusLine(HttpMethodBase.java:1832)
        at org.apache.commons.httpclient.HttpMethodBase.readResponse(HttpMethodBase.java:1590)
        at org.apache.commons.httpclient.HttpMethodBase.execute(HttpMethodBase.java:995)
        at org.apache.commons.httpclient.HttpMethodDirector.executeWithRetry(HttpMethodDirector.java:397)
        at org.apache.commons.httpclient.HttpMethodDirector.executeMethod(HttpMethodDirector.java:170)
        at org.apache.commons.httpclient.HttpClient.executeMethod(HttpClient.java:396)
        at org.apache.commons.httpclient.HttpClient.executeMethod(HttpClient.java:324)
        at net.polyv.jet.encoding.legacy.util.IPUtil.delRemoteServerCacheFile(IPUtil.java:175)
        ***
```

可以看到，这个线程并没有发生死锁，但卡在了发送 HTTP 请求这一步。可能是网络有问题，或者是服务端除了问题，反正我们没收到响应，然后线程就一直停在这了。怎么会这样呢，难道发送 HTTP 请求时没有设置超时时间吗？我一查代码，还真的没设置。。。这是个低级错误啊。

## 3、总结

弄清楚了原因之后，问题就迎刃而解了。总结一下，有几个地方可以改进：

1. 客户端发送 HTTP 请求时，一定要设置超时时间，避免出现问题导致请求卡死。
2. 接收 HTTP 请求的服务端，各级服务器（例如 Nginx、Tomcat）也都要设置超时时间，理由同上。
3. 多线程的程序，出问题时进行排查的难度会相对大一些。所以，对于手工启动、维护的线程，可以的话自定义个线程名称吧，出问题时也有迹可循。

## 4、HttpClient 配置建议

最后，附上一份 HttpClient 配置建议。

由于各种原因，HttpClient 经历过好几次版本变更，且这几次变更导致其 API 用法都不一样，不了解情况的人往往会觉得懵逼，我到底该用哪个版本呢？到底该用哪种方法做配置呢？到底该配置哪几种超时时间呢？下面这个例子，应该基本上涵盖了大多数的应用场景了，拿走不谢。对应的版本是 HttpClient 4.5.* 。

```java
    public static CloseableHttpClient buildHttpClient() throws KeyStoreException, NoSuchAlgorithmException,
            KeyManagementException {
        HttpClientBuilder builder = HttpClientBuilder.create();
        
        // 信任全部 HTTPS 证书，避免 HTTPS 请求因为证书问题而失败。留意，风险自担。
        SSLContext sslContext = new SSLContextBuilder().loadTrustMaterial(null, (arg0, arg1) -> true).build(); // 信任全部证书
        builder.setSSLContext(sslContext);
        HostnameVerifier hostnameVerifier = NoopHostnameVerifier.INSTANCE; // 也信任全部域名
        SSLConnectionSocketFactory sslSocketFactory = new SSLConnectionSocketFactory(sslContext, hostnameVerifier);
        Registry<ConnectionSocketFactory> socketFactoryRegistry = RegistryBuilder.<ConnectionSocketFactory>create()
                .register("http", PlainConnectionSocketFactory.getSocketFactory())
                .register("https", sslSocketFactory).build();
        PoolingHttpClientConnectionManager connectionManager =
                new PoolingHttpClientConnectionManager(socketFactoryRegistry);
        
        // 设置并发连接数上限
        connectionManager.setMaxTotal(CONNECTION_LIMIT_TOTAL); // 总的并发连接数上限
        connectionManager.setDefaultMaxPerRoute(CONNECTION_LIMIT_PER_HOST); // 单个域名的并发连接数上限
        builder.setConnectionManager(connectionManager);
        
        // 设置默认的超时时间。具体数值可按需调整。
        RequestConfig requestConfig = RequestConfig.custom()
                .setConnectTimeout(CONNECT_TIMEOUT) // 建立连接的超时时间
                .setSocketTimeout(SOCKET_TIMEOUT) // 连接建立后，传输数据时的超时时间
                .setConnectionRequestTimeout(CONNECTION_REQUEST_TIMEOUT) // 从连接池中获取连接时的超时时间
                .build();
        builder.setDefaultRequestConfig(requestConfig);
        
        // 除了 GET、HEAD 之外，也自动跟随 POST、PUT 的 301、302 重定向。按需使用。
        builder.setRedirectStrategy(new LaxRedirectStrategy());
        
        return builder.build();
    }
```

参考：
* [HttpClient Timeout](https://www.baeldung.com/httpclient-timeout)
* [How to ignore SSL certificate errors in Apache HttpClient 4.4](http://literatejava.com/networks/ignore-ssl-certificate-errors-apache-httpclient-4-4/)
* 如果需要自动重试机制，可以看 [这里](https://www.jianshu.com/p/580975f49b16)
* 当 POST 遇上302，请看 [这里](https://www.cnblogs.com/cswuyg/p/3871976.html)