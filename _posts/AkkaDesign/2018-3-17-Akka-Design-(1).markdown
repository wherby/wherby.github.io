---
layout: post
comments: true
title:  "Akka design guide (1)   -----How to design a resilient software framework"
date:   2018-3-17 13:22:16 +0800
categories: jekyll update
img: akkadesign.jpg # Add image post (optional)
tags: [akka, resilient, akka design]
---

Let’s begin a journey about how to use Akka actors to build resilient software.

Simple stateless actor:

First with the ActorOne:

```scala
class ActorOne extends Actor with ActorLogging{
  def receive = {
    case Hi => log.info("hi")
    case Crash =>
      log.info("crashing...")
      throw new RuntimeException("Crashed")
    case msg:String=> log.info(msg)
  }
}

```

The Actor will receive 3 types messages, if it’s receive Crash, then the actor will crash with an exception. Let’s test the Actor:

```scala
actone ! Hi
var i = 0
for( i <- 1 to 200){
  actone ! Crash
}
actone ! Hi
actone ! "test"

```

Then the Actor will print Hi message and then crash 200 times and print Hi and “test” as expect.

What’s happened during the test?

1.  First the actor print Hi message as expect
2.  When the actor received Crash message, the actor will Crash, but since the Crash happened in message handling,. Then the actor is recreated and “incarnation” as the previous one, and received the message from the mailbox which will process the message, and crash 200 times for Crash message
3.  Then the actor will process Hi message and “test” message

What’s the test telling?

1.  When a actor is created, even if exception happened in message handle, the actor will handle the rest messages as expected [only if the actor is stateless actor]
2.  If the actor is stateless actor, the actor “acts as” a “service” which all messages sent to actor will be processed in stateless environment
3.  The message is not “sent” to the actor instance, if the actor crashed, the new actor will still process the messages in the actor[incarnation]. In another word, when you send a message to the actor, you sent the message to the ActorRef’s mailbox, only when process the message, the actor context could be determined : the message and message execution is decoupled. 
4.  When design a very complex function, using actor will isolated each message handing, even if some message handling will throw exception, the other function will still work as expected. [So add function to existed actor will have more resilient behavior]



[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
