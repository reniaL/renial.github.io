---
layout: post
title:  "吐槽一下 DDoS 和敏捷开发"
tags: [code]
---

别误会，并不是说敏捷开发模式不好、DDoS 有什么问题，而是说“敏捷开发”、“分布式拒绝服务攻击”这样的名词，容易让人误解。

每次看到“分布式拒绝服务攻击” ([distributed denial-of-service attack](https://en.wikipedia.org/wiki/Denial-of-service_attack)) 这个词，我总觉得有些尴尬，“拒绝”一般是从服务提供者的角度来说的，“拒绝服务”，听起来感觉像是服务提供者 **主动** 做的一件事：我不愿意提供服务了。这样的叫法非常奇怪，举个栗子，王者农药，我们英雄发动技能一刀下去对方掉10%血，然后这个技能叫“喷血攻击”。。。莫名其妙对吧，因为“喷血”类似“拒绝服务”是一个描述自身行为的词，带有主观的意味，从受攻击者的主观角度来描述一个攻击，让人费解。

而更糟糕的是，在“拒绝服务攻击”前面还加上了“分布式”，变成了“分布式拒绝服务攻击”。“拒绝服务”描述的是攻击后服务提供方的状态，而“分布式”描述的是攻击者的行为，两个相反角度的描述糅合在了一起，这谁起的乱七八糟的名字啊，真想对起名的人来一发“圣剑喷血攻击”！随意想到的例如“阻断服务攻击”、“洪水攻击”，都比“拒绝服务攻击”好理解多了，“圣剑冲锋”、“圣剑裁决”听起来也没有“圣剑喷血”那样的不适感对吧，因为这些名字使用的是攻击者的角度（冲锋），或者是第三者的角度（洪水）来描述的。

类似这样起的不恰当的专业名词，时不时都会遇到。起名字的人、资深专业人员可能不以为意，但公众（特别是非专业的）接触到这样的名词时，其实很不好理解。例如“极限编程”、“回归测试”，你如果第一次接触这些词，光看字面肯定不知道它们代表的是什么鬼。当然，起名时要考虑的，不仅仅是“准确”，有时确实难以做到非常准确，这时一般就要求“传神”，例如“精益”，例如“敏捷”。但传神的命名，除了不好理解，还容易把人带偏。

第一眼看到“敏捷软件开发”，会让人想到什么？快！没错，所以“敏捷软件开发 = 快速迭代，灵活应变”是不少人的理解。但实际上，敏捷软件开发宣言所重视的，可远远不止是“快”啊。敏捷宣言包含[4条价值观](http://agilemanifesto.org/iso/zhchs/manifesto.html)，遵循[12条原则](http://agilemanifesto.org/iso/zhchs/principles.html)。这样一套思想、价值观，要用一个词来描述，其实是非常难的。关于敏捷，我个人至今觉得最好的一句话描述，来自 Andy Hunt （敏捷宣言作者之一） 的 *Practices of an Agile Developer* （[高效程序员的45个习惯](https://book.douban.com/subject/4164024/)） ：

> 敏捷开发就是在一个高度协作的环境中，不断地使用反馈进行自我调整和完善。

这个描述很抽象，但我觉得准确地捕捉到了敏捷的两个关键点：**高度协作**，**持续改进**。（从这个角度来看，跟“快”关系不大，叫“敏捷”并不传神。）这样去理解敏捷，甚至可以不局限于软件开发了，我常常觉得，敏捷作为一种价值观、方法论，可以应用于我们的日常生活，特别是个人的成长。每隔一段时间就对自己做个回顾，看看哪些地方做的好、哪些地方做的差，想想怎么改进，接着继续下一阶段的计划。这，不就是我们人生的一个又一个 Sprint 吗~