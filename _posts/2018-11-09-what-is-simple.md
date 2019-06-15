---
layout: post
title:  "何谓简单"
tags: [code]
---

## Lisp 往事

2011年，读《黑客与画家》，其中有一篇《拒绝平庸》，里面说道，“编程就应该使用最强大的语言”。虽然我不完全认同这句话，但这可是出自 Paul Graham 之口啊，他这句话勾起了我对 Lisp 的强烈好奇心。Lisp 号称是最强大的语言，真的这么强吗？将来会不会把 Java C++ 之类的都干掉？我迫不及待地想一探究竟。于是，读完《黑客与画家》之后，我找到了 [Practical Common Lisp](http://www.gigamonkeys.com/book/)，开始学 Common Lisp。

结果证明，Lisp 是我遇到的最难学的语言，我断断续续读完了书的前八章，艰难的学会了关键的 macro，然后就没有然后了。至今想起来，没能把 *Practical Common Lisp* 读完，多少有些遗憾，因为，单纯从编程语言的角度来说，Lisp 的设计实在太独特、太巧妙了，学习这样一种元语言，绝对能开拓人的思路。另外，学会 Lisp 然后拿来装逼也是妥妥的，哈哈。

如今，在7年之后，我又一次有了跟当年类似的感觉：因为一篇文章（准确来说是一个演讲），对一种编程语言产生了浓厚的兴趣，迫不及待地想学这种语言。这个演讲，就是 Rich Hickey 的 [Simple Made Easy](https://github.com/matthiasn/talk-transcripts/blob/master/Hickey_Rich/SimpleMadeEasy.md)。这种语言，就是 Clojure。真巧，又一种 Lisp 方言！

<!--more-->

## Rich Hickey

[Rich Hickey](https://github.com/tallesl/Rich-Hickey-fanclub)，Clojure 作者，知名演讲人，发表过众多编程方面的演讲和文章，其中不少观点被认为对编程的未来将有深远的影响。

认识 Rich Hickey，还得感谢阮一峰。一直有看阮一峰的博文，前不久他写了一篇 [如何降低软件的复杂性？](http://www.ruanyifeng.com/blog/2018/09/complexity.html)，其中提到了 Tcl 语言作者 John Ousterhout 的一本书 [《软件设计的哲学》](https://www.amazon.com/Philosophy-Software-Design-John-Ousterhout/dp/1732102201)，聊了一些软件设计的东西，然后，在阮一峰这篇博文下面，有一个评论是这么说的：

> “关于软件的Simplicity和Complexity，我觉得讲的最好的是clojure的作者Rich Hickey，它是真正洞悉软件复杂度本质的男人，强烈建议阮老师看下他的所有演讲，并且学下clojure，我敢保证clojure会是接下来的百年编程语言。”

“真正洞悉软件复杂度本质的男人”，听起来像不像“要成为海贼王的男人”，哈哈。这个牛逼吹的，可以呀！Clojure 我有所耳闻，但 Rich Hickey 我之前真不认识，真这么牛逼？于是就搜了一下，偶然发现了一个了不起的 GitHub 仓库 [talk-transcripts](https://github.com/matthiasn/talk-transcripts) ！这个仓库收集了一堆跟 Clojure 相关的演讲的 **文字抄本** ，包括 Rich Hickey 的很多演讲都有。其中 [Simple Made Easy](https://github.com/matthiasn/talk-transcripts/blob/master/Hickey_Rich/SimpleMadeEasy.md) 这个演讲很吸引我，因为聊的正是我很想了解的软件复杂度相关的。

## Simple Made Easy

做了十年软件开发，其中一个深刻的感受就是， **我们开发的绝大多数软件系统，都太过于复杂了** 。这里说的包括软件的各个方面——界面设计、产品交互、软件架构、程序实现等等。界面、产品方面我是外行，看法可能有所偏颇。但在程序架构和编码方面，我是有切身体会的。这些年经手过不少软件系统，其中大多数是接手他人的项目，大多数这些项目，从上层的架构设计到底层代码实现，多多少少都有过于复杂的问题。在不断维护、重构这些遗留系统的过程中，我慢慢体会到，简单，在软件设计和开发领域是多么的重要！可惜我们在这方面往往做的还不够好。于是我看 *Simple Made Easy*，想看看从中是不是能学到些东西，结果，真得到了不少启发呢。

## Easy != Simple

文中的一个重要观点是，在软件开发领域，“容易”并不等于“简单”。这两个词有时是可以混用的，但在软件开发的很多情况下，两者确实有所区别。Rich 很有意思，他还从词源这个角度来说明 easy 和 simple 两者的区别。

![00:03:19 Simple](/images/blog/2018-11-09-what-is-simple/00.03.19.jpg)

![00:03:19 Easy](/images/blog/2018-11-09-what-is-simple/00.05.16.jpg)

简单归纳，“容易”是相对的、主观的，反义词是“困难”，“简单”则比较客观，反义词是“复杂”。举个栗子，同一个菜谱，对一位大厨来说可能做起来很容易，但对没怎么下过厨房的小白来说却可能很困难，这跟厨师自身的技能有关。但就菜谱本身来说，我们会说一些菜做起来比较简单，而另一些菜则相对比较复杂，这跟厨师无关，完全就是菜谱自身的一个属性。

除此之外，“容易”还往往表示触手可及的、方便的，易于我们理解，接近我们的技能和能力。而简单，并非只关注维度、数量，其关键点在于没有交错 (interleaving)。

![00:17:14 Development Speed](/images/blog/2018-11-09-what-is-simple/00.17.14.jpg)

这张关于开发速度和关注点（容易还是简单）的关系图，其实是 Rich 自己编出来的，哈哈。他说这是他的感觉，所以并没有具体数值。有意思的是，这让我想起了当年我分享重构时用到的一张图，其实大同小异，其中一个关键点就是，软件系统的维护、开发成本，随着时间递增。另外，Rich 认为，我们在选用一种方案、工具时，往往只留意其好处、用途，而忽略了其副作用、带来的成本。

## “有状态”往往意味着复杂

![State is Never Simple](/images/blog/2018-11-09-what-is-simple/00.35.38.jpg)

Rich 认为，有状态的程序，总是不简单的。有状态的一个例子就是，多次调用一个同一个方法，却可能会得到不同的结果。状态多的话，往往难以重现问题，难以调试。状态不简单，但可能比较容易（留意简单和容易的区别），因为各种东西都很方便获取。但这样往往会把各种东西交织在一起。而且，这里说的有状态，跟并发无关，即使单线程的程序，也是这样。

这其实是个一个老话题了，[有状态 VS. 无状态](https://www.codeproject.com/Articles/834686/Stateful-or-Stateless-classes)，我也一直在思考。我基本上认同 Rich 的观点，认为无状态的程序更易于维护，可能其中一个原因是我慢慢习惯了 Spring 容器管理 bean 的风格？另外还有一个佐证就是，Martin Fowler 的《重构》中有提到一种重构方法，就是将太过于复杂的的 method，先提取为 "method object"，再慢慢的、一步一步的重构。而所谓 method object，往往就是带有状态的对象。为什么要这样重构呢？为什么要有状态呢？不就是因为原始方法实在太过于复杂了嘛。

那么，要如何消除状态呢？Rich 认为，将有状态的程序改为一个 functional interface，确保相同的输入，会得到相同的输出，这是唯一的办法。其它的诸如模块化、封装等方法，都难以去除状态。

然后 Rich 提到 "Clojure refs compose value and time"，因为我没学过 Clojure ，没完全看懂这句话。意思好像是，Clojure refs 不仅包含了值，也有时间的含义在里面？我看到这里时，就觉得，冲着这么有意思的特性，都必须去学一学 Clojure 啊！

## 如何实现简单的设计

Rich 写过 Java，写过 C#，设计了 Clojure，做过大型系统。他相信，复杂的、大型的系统，都可以通过简单得多的工具来实现。他把程序中的各种常规的结构、用法，都怼了一遍，说每种结构、用法都会使一些东西交织在一起，都导致了复杂。然后逐一说明了可以用什么更好的方法、更简单的工具来实现同样的功能。

![The Complexity Toolkit](/images/blog/2018-11-09-what-is-simple/00.39.28.jpg)

![The Simplicity Toolkit](/images/blog/2018-11-09-what-is-simple/00.42.55.jpg)

Rich 引用了 Edsger Dijkstra 的一句话，来表达一个观点：编程事关思考，而非简单的打字。这点我同意，以前就在哪里看到过这样一句话：敲一周代码，能节省你三小时的思考时间。需要思考啥呢？Rich 认为，要设计出简单的东西，关键就是编程时想清楚几个基本的方面： who, what, when, where, why, and how 。很基本的信息，是吧，弄清楚它们，并且别互相交织在一起，这就是实现简单设计的起点。

最后，Rich 说，简单是一种选择。注意，简单并非容易、就手。要实现简单的设计，必须保持时刻警惕，必须对交织在一起的东西有敏感性，而这需要慢慢培养起来。

## 参考资料

- [Simple Made Easy](https://github.com/matthiasn/talk-transcripts/blob/master/Hickey_Rich/SimpleMadeEasy.md)
    - ↑ 图片来源 ↑
    - 这个仓库的作者是个自由职业全栈开发者，[他觉得](http://matthiasnehlsen.com/blog/2014/10/15/talk-transcripts/)，跟看视频相比，他更倾向于通过读文字 (reading) 来学习。深有同感啊，握爪！
- [如何降低软件的复杂性？](http://www.ruanyifeng.com/blog/2018/09/complexity.html)
