---
layout: post
comments: true
title:  "Asking questions"
date:   2017-12-23 13:22:16 +0800
categories: jekyll update
img: Ask.jpg # Add image post (optional)
tags: [scala, play, stream, micro service, akka]
---

The story begins from a question I asked when I develop Hydra, I found the WS library become complex after play 2.6.X, so I asked the question in stackoverflow:
[https://stackoverflow.com/questions/46683070/ws-in-play-become-incredible-complex-for-2-6-x](https://stackoverflow.com/questions/46683070/ws-in-play-become-incredible-complex-for-2-6-x)




![questions](/media/AskQuestion/ask1.jpg)


The question is that before Play 2.6.x the WS library is already async I/O, we could easily use the feature as below [https://engineering.linkedin.com/play/play-framework-async-io-without-thread-pool-and-callback-hell](https://engineering.linkedin.com/play/play-framework-async-io-without-thread-pool-and-callback-hell): 

```scala
    val responseFuture: Future[Response] = WS.url("http://example.com").get()
```


And before Play 2.4, when using play framework, the Akka related feature is not explicit used, the play is very fast framework at that time. 

When we dig more about the question: 

 1.  Why Akka related feature is added to Play
 2.  What’s benefit of the change
 3.  What’s the change will impact the Play

First thing first, why Akka related feature is added to Play? Because they can 😊, Akka and Play are running by same company: Lightbend[Typesafe]. Before Play 2.4, play is using the Non-Block Http server [https://www.lightbend.com/blog/why-is-play-framework-so-fast](https://www.lightbend.com/blog/why-is-play-framework-so-fast), which is fast and basically thread pool related.

![playserver](/media/AskQuestion/playserver.jpg)

And at the same time, Akka is developing Akka-http server which will provide “STREAMED” Http services[https://doc.akka.io/docs/akka-http/current/index.html](https://doc.akka.io/docs/akka-http/current/index.html). Because Akka-http is based on TCP layer[https://doc.akka.io/docs/akka-http/current/server-side/low-level-api.html?language=scala](https://doc.akka.io/docs/akka-http/current/server-side/low-level-api.html?language=scala), then performance is much greater than play http server. After the Akka-http develop, the feature is gradually added to Play.

Of course, the benefit is the speed of the response time. The popular micro-service need the “streamed” http server.  

What’s will impact the Play? I think Akka-http will be co-existed with Play Netty for much long time[https://www.playframework.com/documentation/2.6.x/Highlights26](https://www.playframework.com/documentation/2.6.x/Highlights26), and use Akka-http as default server. But as the original question, some trivial feature like WS library is replaced.


# BTW Lightbend also provide simple way to use native Akka stream feature in Spring:
[https://akka.io/blog/news/2017/10/23/native-akka-streams-in-spring-web-and-boot-via-alpakka?utm_content=buffer7c1f6&utm_medium=social&utm_source=linkedin.com&utm_campaign=buffer](https://akka.io/blog/news/2017/10/23/native-akka-streams-in-spring-web-and-boot-via-alpakka?utm_content=buffer7c1f6&utm_medium=social&utm_source=linkedin.com&utm_campaign=buffer)


Pic from http://www.incidentalcomics.com/2015/08/asking-questions.html:

![askquestion](/media/AskQuestion/AskPic.jpg)

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
