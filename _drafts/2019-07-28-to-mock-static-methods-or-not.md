---
layout: post
title:  "静态方法，mock 还是不 mock，这是个问题"
tags: [code]
---

不知从何时开始，Mockito 成了 Java 的单元测试框架王者，目前（2019年7月）Github 上 star 数直逼 10K。看看其他的单元测试工具：PowerMock 2K（无疑是沾了 Mockito 的光），easymock 600，JMockit 300。跟 Mockito 一比，好可怜啊，一个能打的都没有。

Mockito 当然很好。我从2012年还是2013年开始用 Mockito，看着它从 1.0x 版本一路走来，今年晚些时候估计会正式发布 3.0 版本。应该有不少人都跟我有类似的体验，从 Mockito 开始接触 mock / stub，一边赞叹 Mockito 语法的简练，一边享受着 mock 带来的单元测试的便利性。总说单元测试应该要隔离外部依赖和实现，很难想象，如果没有 mock，怎么写单元测试呢？

```java
public void test() {
    when(userDao.update(any(User.class))).thenReturn(1);
    int actual = userService.update(aUser);
    Assert.assertTrue(acutal > 0);
    verify(userDao).update(aUser);
}
```

看看上面这个 Mockito 的例子，when(...).thenReturn(...)，verify(...).doSomething()，这代码就像人类语言，多么简明易懂啊！

但是（没错转折来了），已经2019年了，Mockito 依然不支持 mock 静态方法、构造方法等。你可以说，这是设计理念，Mockito 首页上一直写着一句话 "Don’t mock everything"，认为说应该做好功能代码的设计，尽量避免静态方法等，尽量使你的代码易于测试。这个理念，在理论上没问题，但这么多年的开发经验告诉我，理想归理想，实际上要你去维护的遗留代码总是一箩筐一箩筐的，避无可避。

> To mock static methods or not to mock, that is a question.

单元测试中是否要 mock 静态方法，一直争论不休，看看 [StackOverflow](https://stackoverflow.com/questions/4482315/why-doesnt-mockito-mock-static-methods) 上这个讨论就知道，各种意见都有。真要用 Mockito 来 mock 静态方法，一般都是结合 PowerMock 使用。这两年 PowerMock 发展的怎么样我不太清楚，但14、15年那会儿我用过 PowerMock，感受就是，真他妈累啊！理论上来说是可以的，但实际做起来就总是各种问题，然后各种 google 、解决，然后又继续各种问题，排查的我都快怀疑人生了。最终我是放弃了 PowerMock 的，这么费力地去结合两个工具一起用，往后很难说还有多少坑。

Mockito、EasyMock 等工具不支持 mock 静态方法，原理上是因为它们都是基于 cglib 的，只能通过创建子类或实现接口的方式去 mock。而 JMockit 则不一样，使用 JDK6 的 java.lang.instrument 包和 ASM，动态地在运行时修改字节码，从而实现 "Mock Anything" 。一个 JMockit ，解决了 Mockito + PowerMock 两个工具都解决不了的问题，那为啥不用 JMockit 呢？