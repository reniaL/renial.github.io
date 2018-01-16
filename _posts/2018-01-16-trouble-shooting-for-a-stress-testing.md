---
layout: post
title:  "记一次压力测试问题排查过程"
tags: [code]
---

## 1、初始现象

用于测试的架构：一台 Nginx，upstream 为一至三台 Tomcat，Tomcat 应用连接了两套 Redis （由于历史遗留原因。。。）：一套是3主3从的 Redis 集群，一套是1主1从的单节点 Redis。

机器：被测机器都是物理机，CPU 都是16核或24核以上，内存 32G ~ 128G，千兆网卡。

测试方法：一台压力机，使用 ab 测试 Nginx 的动态页面，一次页面请求大概会查询10来次 Redis，没压力时响应时间一般在500ms以下。

【测试结果】：Nginx 带一台 Tomcat 时，QPS 为 2500 ~ 3000；Nginx 带两台 Tomcat 时，QPS 为 3000 ~ 3300；三台 Tomcat 时，跟两台 Tomcat 一样。

机器表现：Nginx 和 Tomcat 机器的 CPU 负载不高，30%左右。Redis 机器负载较低。从监控上看带宽也没到瓶颈。

## 2、发现 Spring Session 的问题

一开始，怀疑是 Spring Session 和单节点 Redis 的问题，尝试将单节点 Redis 改为集群。

【测试结果】：单台 Tomcat 的 QPS 有小量提升，一台 Nginx + 两台 Tomcat 的 QPS 没有改善。

继续排查，发现了 Spring Session + Spring Data Redis 的一些问题，这篇文章里详细说清楚了：
https://medium.com/@odedia/production-considerations-for-spring-session-redis-in-cloud-native-environments-bd6aee3b7d34

于是升级了 Spring Session 版本，然后改了一些配置，主要是禁用 Redis 的事件通知机制，避免 Tomcat 数量越多，对 Redis 的压力越大。再后来甚至试了完全禁用了 Spring Session。

【测试结果】：Nginx 的 QPS 没有大变化。

## 3、折腾 Redis

排查应用的日志发现，高并发时做 Redis 查询可能会比较耗时。于是怀疑是达到了单节点 Redis 的 QPS 瓶颈。使用 redis-benchmark 测试，单台 Redis 的 QPS 为 6w+。尝试将应用中的 Redis 查询全部迁移到 Redis 集群上。

测试结果：Nginx 的 QPS 没有大变化。单台 Redis 的 QPS 可以达到1w+，貌似并没达到瓶颈。

尝试去除请求中后端的 Redis 查询，发现 Nginx 的 QPS 可以达到 1w+。尝试减少一次请求中的 Redis 查询次数，尝试用 Redis Pipelining，发现目前 Jedis (2.9.0) 集群不支持 Pipelining。。。

怀疑是 Redis 集群所在的网络问题，尝试更换 Redis 节点、集群，尝试调整 Redis 客户端连接池配置，尝试调整 Redis 机器的系统配置等等。。。

【测试结果】：木有变化。

## 4、LVS, Nginx, Tomcat 各种组合尝试

实在没办法，只能各种尝试。首先试了跳过 Nginx 直接测试Tomcat。

【测试结果】：两台压力机，每台压一个Tomcat，每个Tomcat的QPS都能去到接近 3000。

这个结果有些进展，怀疑是 Nginx 配置问题，跃跃欲试地试了 LVS 带两台 Tomcat，结果。。。

【测试结果】：LVS 带 Tomcat，与 Nginx 带 Tomcat 的情况一样有问题。

这个时候，我们已经开始怀疑人生了。

## 5、jstack 及其他

在一个友军的帮助下，我们在压测时用 jstack 把 Tomcat 的线程信息 dump 了出来，查看这些线程，发现有很多的线程在等待，几千个，类似这样：

```js
"http-nio-8080-exec-4042" #4094 daemon prio=5 os_prio=0 tid=0x00007f53ec05e000 nid=0x3687 waiting on condition [0x00007f56949c8000]
   java.lang.Thread.State: WAITING (parking)
    at sun.misc.Unsafe.park(Native Method)
    - parking to wait for  <0x00000005c2d7bc80> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
    at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
    at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:103)
    at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:31)
    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1067)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1127)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
    at java.lang.Thread.run(Thread.java:745)
```

一度怀疑是哪里的线程池阻塞了，赶紧一番的 Google ，最终在 [StackOverflow](https://stackoverflow.com/questions/8086463/finding-the-cause-for-waiting-sleeping-threads) 上查到，这些线程应该只是 Jedis 的空闲线程池，不是阻塞。还是无功而返。

随后我一度怀疑会不会是 ab 的问题（虽然我自己也不太相信。。。），于是另外找了个工具叫 wrk 的来代替 ab 做测试。

【测试结果】：木有变化。

## 真·答案

几个排查的人都没辙了，大家都靠自己的直觉在继续排查和尝试。我总觉得，真相应该藏在某个并不复杂的角落里。而且我对之前“两台压力机分别压两个Tomcat”的结果耿耿于怀，于是用一些关键字在 Google：apache benchmark, throughput, network... 然后，偶然翻到这篇 [StackOverflow](https://stackoverflow.com/questions/596590/how-can-i-get-the-current-network-interface-throughput-statistics-on-linux-unix) ，里面说到一个工具 iftop。于是在一台 Tomcat 上装了，运行，测试，一看，我靠，带宽直逼 1G ！赶紧拉运维同学过来看，一番讨论，终于找到罪魁祸首：带宽跑满了，之前我们看监控上的带宽数据，因为算法问题被平均了，所以一直没发现。。。

像很多故事一样，最终发现的问题原因，其实很简单，真的很简单。但是却折腾了我们几天时间，很久没有像这样为一个技术问题折腾好几天了。不过，我觉得这几天时间也并没有浪费，这个排查的过程，学到了很多新东西，例如 Spring Session 的一些问题，例如 Redis 的一些配置和用法，例如各种工具 (jstack, ab, wrk, iftop)，等等。