---
layout: post
comments: true
title:  "The Tao in web service"
date:   2018-1-19 13:22:16 +0800
categories: jekyll update
img: taoofservice.png # Add image post (optional)
tags: [tao, scala, akka, akka-http, streaming, fast data, fast api]
---

Let’s begin with a pic snap from [https://www.youtube.com/watch?v=h3mulWmX1Oo](https://www.youtube.com/watch?v=h3mulWmX1Oo) as below :

![cpu cost](/media/TaoOfWebservice/cpu.png)


The surprising fact is that the thread context switch operation needs about 10000 to 10000000 times CPU Cycles, and what’s more surprising fact underneath of the fact is that our mainstream web server is based on thread context switch [Threading pool].

Let’s begin from extreme simple web service of add service which accept two inputs and return sum. If the service running on a host with 2.4G Hz CPU, the host could do 2.4 * 10^9 times adding operation but could only handle at most 2.4 * 10^5 web request of add operation web service [For each web request will trigger thread context switch which costs 10000 CPU Cycles at least]. If a web server could handle 10^5 request per second, that’s must be an awesome server with a well-designed application, but in the article, only consider two factors: the thread context switch and the operation in web service.

The thread context switch is the overhead in the webservice. How to minimize time on thread context switch and make the application faster?
1.  Make 10 times faster?
2.  Make 100 times faster?

# How to make 10 times faster?
Simple task first, how can you makes a web application 10 times faster?
The following words from intel blog [https://software.intel.com/en-us/node/506100](https://software.intel.com/en-us/node/506100):

![intel](/media/TaoOfWebservice/intel.png)

If the web application treats the web requests as tasks rather than threading job, the performance will gain 18 times speed up at least. 

“Talking is cheap, show me the code!” Is there any web server treats web request as task?
See the following pic from [https://akka.io/blog/2016/07/06/threading-and-concurrency-in-akka-streams-explained](https://akka.io/blog/2016/07/06/threading-and-concurrency-in-akka-streams-explained)

![thread in akka](/media/TaoOfWebservice/threadingInAkka.png)

The Akka Http Streaming will treat the web request as tasks, for example, if ten different clients call the sum service written in Akka-Http, there will only one thread handle these requests. 

Therefor using streamed web service [Flow package in Java and Akka-Http] will makes your web server’s thread context switch 10 times faster.

# How to make 100 times faster? 

Take a nap and have a dream about it😊! 

In the Tao’s world, image the web request as an action in Spark, the web requests are the Source, then the web application is the action, and the web response is the Sink of the flow. Then there will gain more than 100 times faster jobs.

# Use cases

So why there is need for speeding up web server response time? 

## For micro service 
When existed service breakdown to micro service, there will have multiple times costs in web server response because from one web request call to multiple request calls. Speed up the web response time will greatly speed up micro service.

## For IoT structure
In IoT world, the web application will handle trivial web requests which will only little time for business logic. It is important to speed up the server side response time.

## For any web services
It’s always better to have quicker server. 




[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
