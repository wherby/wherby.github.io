---
layout: post
comments: true
title:  "The Tao of web service"
date:   2018-1-19 13:22:16 +0800
categories: jekyll update
img: taoofservice.png # Add image post (optional)
tags: [tao, scala, akka, akka-http, streaming, fast data, fast api]
---

Let’s begin with a pic snap from [https://www.youtube.com/watch?v=h3mulWmX1Oo](https://www.youtube.com/watch?v=h3mulWmX1Oo) as below :

![cpu cost](/media/TaoOfWebservice/cpu.png)


The surprising fact is that the thread context switch operation needs about 10000 to 10000000 times CPU Cycles, and what’s more surprising fact underneath of the fact is that our mainstream web server is based on thread context switch [Threading pool].

Let’s begin from extreme simple web service of add service which accept two inputs and return sum. If the service running on a host with 2.4G Hz CPU, the host could do 2.4 * 10^9 times adding operation but could only handle at most 2.4 * 10^5 web request of add operation web service [For each web request will trigger tread context switch which costs 10000 CPU Cycles at least]. If a web server could handle 10^5 request per second, that’s must be an awesome server with a well-designed application, but in the article, only consider two factors: the thread context switch and the operation in web service.

The thread context switch is the overhead in the webservice. How to minimize time on thread context switch and make the application faster?
1.  Make 10 times faster?
2.  Make 10000 times faster?


[To Be Continue…]

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
