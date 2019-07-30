---
layout: post
title:  "静态方法，mock 还是不 mock，这是个问题"
tags: [code]
---

## 王者 Mockito

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

看看上面这个 Mockito 的例子，when(...).thenReturn(...)，verify(...).doSomething()，这代码就像人类语言，多么简明易懂！

但是（没错转折来了），已经2019年了，Mockito 依然不支持 mock 静态方法、构造方法等。你可以说，这是设计理念，Mockito 首页上一直写着一句话 **"Don’t mock everything"** ，认为说应该做好功能代码的设计，尽量避免静态方法等，尽量使你的代码易于测试。这个理念，在理论上没问题，但这么多年的开发经验告诉我，理想归理想，实际上要你去维护的遗留代码总是一箩筐一箩筐的，避无可避。

## Static methods, to mock or not to mock, that is a question

单元测试中是否要 mock 静态方法，一直争论不休，网上有 [一个](https://stackoverflow.com/questions/4482315/why-doesnt-mockito-mock-static-methods) [一个](https://testing.googleblog.com/2008/12/static-methods-are-death-to-testability.html) [又一个](https://github.com/mockito/mockito/issues/1013) 的讨论，各种意见都有。

我的个人意见，跟 [这个观点](https://stackoverflow.com/questions/4482315/why-doesnt-mockito-mock-static-methods#comment18043894_4482364) 一样，我认为测试工具不应该替用户决定什么是好、什么是不好，而应该尽量提供选择，让用户自行判断、采取合适的方案。理论很美好，但实际情况就是，google 搜 "mockito how to mock static methods"，有近15万条结果，可想而知，全世界的开发者在这个问题上浪费了多少时间。

真要用 Mockito 来 mock 静态方法，一般都是结合 PowerMock 使用。这两年 PowerMock 发展的怎么样我不太清楚，但14、15年那会儿我用过 PowerMock，感受就是，真他妈累啊！理论上来说是可以的，但实际做起来就总是各种问题，然后各种 google 、解决，然后又继续各种问题，排查的我都快怀疑人生了。最终我是放弃了 PowerMock 的，这么费力地去结合两个工具一起用，往后很难说还有多少坑。

Mockito、EasyMock 等工具不支持 mock 静态方法，原理上是因为它们都是基于 cglib 的，只能通过创建子类或实现接口的方式去 mock。那除了 cglib ，就没有其他的 mock 实现方法了吗？当然有，修改字节码呀！

## 另辟蹊径的 JMockit

和其他大多数使用 cglib 实现的单元测试工具不同，JMockit 使用 JDK6 的 java.lang.instrument 包和 ASM，动态地在运行时修改字节码，从而实现 **"Mock Anything"** 。什么静态方法、构造函数，随时随地想 mock 就 mock。一个 JMockit ，解决了 Mockito + PowerMock 两个工具都解决不了的问题，那为啥不用 JMockit 呢？JMockit 为啥流行不起来呢？

```java
public class UserServiceTest {

    @Tested
    private UserService userService;
    @Injectable
    private UserDao userDao;

    public void test() {
        new Expectations() {
            {
                userDao.update(withInstanceOf(User.class));
                result = 1;
            }
        };

        int actual = userService.update(aUser);
        Assert.assertTrue(acutal > 0);

        new Verifications() {
            {
                userDao.update(withInstanceOf(User.class));
            }
        };
    }
}
```

功能更强大的 JMockit 却流行不起来，我觉得其中一个原因，是它的语法不太友好。看看上面这个 JMockit 的例子，这坨 new Expectations(){...} 和 new Verifications(){...} 是什么鬼？匿名类？为啥里面又有一层大括号？别说测试代码了，在普通的功能代码中，我们都极少见到这样的语法。多数人可能觉得不习惯，然后就此打住，放弃 JMockit 了。

JMockit 的这种语法，是基于它的 [record-replay-verify](https://jmockit.github.io/tutorial/Mocking.html#model) 模型。new Expectations() 是录制期望，new Verifications() 是校验，二者中间的就是回放——正常调用业务方法。而在匿名内部类类中间的那层大括号，是 Java 的“实例初始化块” (Instance Initialization Blocks)，我们平时可能用“静态初始化块”比较多，“实例初始化块”确实较少见，它的其中一种用途，就是用来初始化匿名内部类，因为匿名内部类不能有构造函数。理解了这些语法之后，其实 JMockit 不难懂，用法跟其他测试框架也大致一样，就是功能更强大了。

JMockit 不够流行的另一个原因，我猜可能跟社区有关。没办法，Mockito 太受欢迎了，社区一片火热，贡献者一大堆。反观 JMockit，虽然开源，但只有原作者 [Rogério Liesenfeld](https://github.com/rliesenfeld) 自己一个人在开发维护。这种单人维护的项目，说不定哪一天就停更了，大家都会有这种担忧。我也担心啊，但看看近几年 JMockit 的 [release notes](https://jmockit.github.io/changes.html)，基本上固定每一、两个月一次发布，并且还会提前订好下一次发布的计划，真想对作者说一句：老哥，稳！所以，至少目前看来，JMockit 的稳定性、活跃性是不用担心的，毕竟有个这么稳的作者。

## JUnit5 + JMockit + Surefire + Jacoco 的配置例子

想要安心用上 Junit5 和 JMockit，还想要单元测试覆盖率？那还是有些坑要踩的。以 Maven 为例，有几个留意点：

* 默认的 maven-surefire-plugin （运行单元测试的） 的版本不支持 JUnit5，得手动指定新版本号才行。
* 想用 jacoco-maven-plugin 得到单元测试覆盖率的话，因为 jacoco 也用了修改字节码的方案，默认配置下会和 JMockit 有冲突。需要一些额外配置才行。

完整的 Maven 配置例子：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>io.github.renial</groupId>
    <artifactId>java-utils</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <name>java-utils</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>

        <jmockit.version>1.46</jmockit.version>
        <jupiter.version>5.4.2</jupiter.version> <!-- 别用 junit.version！否则可能会影响其他使用 junit4 库 -->
        <surefire.version>2.22.2</surefire.version> <!-- 指定版本，以支持 JUnit5 -->
        <jacoco.version>0.8.4</jacoco.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.jmockit</groupId>
            <artifactId>jmockit</artifactId>
            <version>${jmockit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${jupiter.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${surefire.version}</version>
                <!-- 分别指定 jmockit 和 jacoco 两个 agent，以支持运行 jmockit 测试，支持 jacoco 的覆盖率 -->
                <!-- 可参考 https://github.com/jacoco/jacoco/issues/193 -->
                <configuration>
                    <argLine>
                        -javaagent:${settings.localRepository}/org/jmockit/jmockit/${jmockit.version}/jmockit-${jmockit.version}.jar -javaagent:"${settings.localRepository}"/org/jacoco/org.jacoco.agent/${jacoco.version}/org.jacoco.agent-${jacoco.version}-runtime.jar=destfile=${project.build.directory}/jacoco.exec
                    </argLine>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>${jacoco.version}</version>
                <executions>
                    <!-- prepare-agent 这个 goal，原本的效果就是带上 -javaagent:*** 参数，以指定 jacoco agent -->
                    <!-- 但是，maven-surefire-plugin 手工配置了 jacoco 的 agent 之后，这里的 prepare-agent 实际上不会生效 -->
                    <!-- 不过，maven-surefire-plugin 中引用的 jacoco jar 包，需要运行一次该 prepare-agent 的 goal 之后才有 -->
                    <!-- 所以，还是配置上吧，没啥坏处 -->
                    <execution>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>report</id>
                        <phase>test</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```