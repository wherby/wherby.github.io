---
layout: post
comments: true
title:  "Gig introduction"
date:   2018-09-24 13:22:16 +0800
categories: jekyll update
img: gig.jpg # Add image post (optional)
tags: [gig,kafka,akka]
---

Let's start from the beginning: 

If you want to consume messages from kafka, you need to integrate with kafka connector: 
Use the [scala-kafka-client](https://github.com/cakesolutions/scala-kafka-client) as an example.

 ```Scala

    val localKafkaServer= new KafkaServer(kafkaPort.toInt)
    localKafkaServer.startup()
    Thread.sleep(199)
    implicit val system = ActorSystem("test")
    val downStreamTestActor = system.actorOf(Props[DownStreamTestActor],"downstreamActor")
    val consumer= Consumer.createConsumer(downStreamTestActor)
    val producer = GigProducer.createProducer()
    val topicPartition = randomTopicPartition

    //Send first message to topic
    producer.send(KafkaProducerRecord(topicPartition.topic(), None, "value1"))
    producer.flush()
    val subscription = Subscribe.AutoPartition(List(topicPartition.topic()))
    //Create a donwstream actor to consume message in one topic
    consumer.subscribe(subscription)                          
    Thread.sleep(10000)
    //Send another message to topic
    producer.send(KafkaProducerRecord(topicPartition.topic(), None, "value2"))
    producer.flush()
    //Create another downstream actor to consume same topic 
    val downStreamTestActor2 = system.actorOf(Props[DownStreamTestActor],"downstreamActor2")
    val consumer2= Consumer.createConsumer(downStreamTestActor2)
    consumer2.subscribe(subscription)
    Thread.sleep(10000)
    //Unscribe the fist actor
    consumer.unsubscribe()
    Thread.sleep(60000)

```

In the code:
1.create a downstream actor to consume a topic from kafka server, but the actor will not send back ACK message to connector.
2.Then we add another actor to listen to same topic
3.After some time, we unsubscribe the first actor

 ```

2018-09-24 12:21:43,448 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - Yo, start  new downstream receiver 
2018-09-24 12:21:53,161 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - Offsets(XUsjn-0 = 1)
2018-09-24 12:21:53,161 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - value1
2018-09-24 12:21:55,442 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - Yo, start  new downstream receiver 
2018-09-24 12:21:56,238 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - Offsets(XUsjn-0 = 1)
2018-09-24 12:21:56,238 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - value1
2018-09-24 12:21:57,270 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - Offsets(XUsjn-0 = 1)
2018-09-24 12:21:57,270 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - value1
2018-09-24 12:22:00,294 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - Offsets(XUsjn-0 = 1)
2018-09-24 12:22:00,294 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - value1
2018-09-24 12:22:05,332 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - Offsets(XUsjn-0 = 1)
2018-09-24 12:22:05,332 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - value1
2018-09-24 12:22:15,404 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - Offsets(XUsjn-0 = 2)
2018-09-24 12:22:15,404 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - value1
2018-09-24 12:22:15,404 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - value2
2018-09-24 12:22:18,457 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - Offsets(XUsjn-0 = 2)
2018-09-24 12:22:18,457 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - value1
2018-09-24 12:22:18,457 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - value2
2018-09-24 12:22:21,511 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - Offsets(XUsjn-0 = 2)
2018-09-24 12:22:21,511 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - value1
2018-09-24 12:22:21,511 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - value2
2018-09-24 12:22:26,534 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - Offsets(XUsjn-0 = 2)
2018-09-24 12:22:26,534 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - value1
2018-09-24 12:22:26,534 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor2 - value2

```


From the log, we find the behavior of the connector:
1. If the actor not send ACK message back, the connector will resend several times to the actor
2. If resend messages will not be ACKed, then the connector will wait for ACK or the unsubscribe message
3. If one message not ACKed, the connection of the topic will be blocked which is not desired behavior in the fast system


How could we build a better connector based on the previous connector?

1. Could the connector not use ACK?
No, because the ACK message is used to confirmation from consumer. While the consumer actor is implimented by consumer which
may involve some dangerous operation, we could isolate the consumer operation with ACK operation.

2. Do we need retry for same message?
Not nessary, if some message failed, better way is to retry in another time or within another context.

Based on the two principles, the Gig is created:
1. There will have a  ConsumerActor in connector, the actor will send ACK messages based on reply of downsteam Actor:
   a. If downstream actor sends ACK to ConsumerActor, then the message is ACKed
   b. If downstream actor does not send ACK to ConsumerActor, then the message will timeout, but the message will still be ACKed
   c. In one batch messages from Kafka, 
        if the message is ACKed, then the message will flowed to ACKed Actor(which is user implemented) to handle
        if message is not ACKed, the message will flowed to NoACKed Actor(which is user implemented) to handle.   

2. User could implement NoAcked Actor to handle the not Acked message instend of resend message

How about the performance of the Gig?

```
2018-09-24 13:34:20,707 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - value 0
2018-09-24 13:34:20,707 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - value 99
2018-09-24 13:34:20,707 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - value 199

.....


2018-09-24 13:34:25,050 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - value 4999649
2018-09-24 13:34:25,050 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - value 4999749
2018-09-24 13:34:25,050 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - value 4999849
2018-09-24 13:34:25,050 INFO  t.g.d.DownStreamTestActor akka://test/user/downstreamActor - value 4999949


```

The Gig consumer will consume 1000000 message per second. If you want more throughput, just add more partition for the topic.


Github of Gig: [gig](https://github.com/wherby/gig)


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
