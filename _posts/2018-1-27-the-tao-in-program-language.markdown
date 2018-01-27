---
layout: post
comments: true
title:  "The Tao in program language [Chinese]"
date:   2018-1-27 13:22:16 +0800
categories: jekyll update
img: taoofprogram.jpg # Add image post (optional)
tags: [tao, program language,  akka, java, concurrent]
---

Let's begin with a java book. which tells how to create thread-safe code in concurrent programing. 


这篇就用中文了，惊不惊喜，意不意外。

前几天刚去新公司入职，碰巧遇到了上海N年不遇的大雪，我的电脑也碰巧还在顺风那里。所以第一个早上就在座位上浏览了同事带来的这本书了。



![JavaBook](/media/TaoOfProgram/java.jpg)

这本书的确是一本非常好的Java多线程的指导书，书中深入浅出地介绍了多线程设计中会遇到的线程安全问题，用Java自己带的各种机制去解决多线程带来的竞争和各种巧妙的设计避免了各种锁带来的性能下降问题。

下面由我来简单模拟分析一下这本书解决的问题：

【问题】
因为

    a. 程序很多任务是同步的任务（需要等待外部操作）资源

    b. 要对不同的使用者公平

    c. 可以让程序按照功能拆分并能协调合作

这些原因如下图所示（snap on page 1）就是为了解决了让程序更快响应用户的需求的条件下使用了多线程编程的模式

![multithread](/media/TaoOfProgram/multithread.png)

【解决方案一】：

    a. 由于使用了多线程编程的模式，导致了出现多线程的竞争等等，所以Java就引入了各种同步原语和锁机制保证了线程的安全性

    b. 由于各种线程的各种同步原语和锁会引发死锁和饥饿等不好的情况，所以这本书就给出了很多办法去缓解这些症状。

在Java 的世界里，这个书的确是对问题做了比较好的解答， 全书一共200多页，你要按照书里的方案去避免各种显示和隐式的陷阱得到一个比较好的结果。 但是有没有更好的解决方案呢？ 


【解决方案二】： 

    a.  对任何资源访问限制为单一线程访问，解决了线程竞争的问题，就不用使用同步原语和锁

    b.  对外部响应采用Reactive 机制，解决了公平性问题

    c.  系统分为各个部分，通信采用消息模式，解决了便利性的问题

这个解决方案又有一个名字叫[Akka](https://akka.io/) 😊

![akka](/media/TaoOfProgram/akka.png)

那怎么从Java的角度理解Akka解决的问题呢？

在Java世界中，码农们需要自己去管理ThreadPool还有各种底层的资源，所以码农们身上就应该有更大的责任把这些资源合理分配好。 而Akka 则用了一些办法（详见Akka文档）让更专业的人员去负责设计调度那些关键的底层资源，码农们就可以更加专注于业务逻辑。
【With great power comes great responsibility】专业的事情交给专业的人做。

[To be continue ...]

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
