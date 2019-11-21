---
layout: post
comments: true
title:  "Scala priority queue with in the order dequeue"
date:   2019-11-21 13:22:16 +0800
categories: jekyll update
img: priorityqueue.jpg # Add image post (optional)
tags: [scala, pattern match]
---

Scala library has priority queue implementation, but when priority are equal the dequeued items are not in the same order with enqueue order:

``` scala

import scala.collection.mutable

class PriorityQueueTest{
 implicit val ord: Ordering[(Any,Int)] = Ordering.by(_._2)

 var queue = mutable.PriorityQueue[(Any,Int)]()

}

val pQueue = (new PriorityQueueTest())
val queue = pQueue.queue

queue.enqueue(("a",3))
queue.enqueue(("b",3))
queue.enqueue(("c",3))
queue.enqueue(("d",3))
queue.enqueue(("e",3))
println(queue)
//PriorityQueue((a,3), (b,3), (c,3), (d,3), (e,3))
queue.dequeue()
//res6: (Any, Int) = (a,3)
println(queue)
//PriorityQueue((e,3), (b,3), (c,3), (d,3))
queue.dequeue()
//res8: (Any, Int) = (e,3)
println(queue)
//PriorityQueue((d,3), (b,3), (c,3))

```

How to get the dequeue with same order:
``` scala

import scala.collection.mutable

class PriorityQueueTest{
 implicit val ord: Ordering[(Any,Int)] = Ordering.by(_._2)

 var queue = mutable.PriorityQueue[(Any,Int)]()

 def inplaceDeque(number:Int): Seq[(Any,Int)] ={
  var res: Seq[(Any,Int)] = Seq()
    res =queue.take(number).toSeq
    queue =queue.drop(number)
  res
 }
}

val pQueue = (new PriorityQueueTest())
val queue = pQueue.queue

queue.enqueue(("a",3))
queue.enqueue(("b",3))
queue.enqueue(("c",3))
queue.enqueue(("d",3))
queue.enqueue(("e",3))
println(pQueue.queue)
// PriorityQueue((a,3), (b,3), (c,3), (d,3), (e,3))
pQueue.inplaceDeque(1)
// Seq((a,3))
println(pQueue.queue)
// PriorityQueue((b,3), (c,3), (d,3), (e,3))
```

# Question
1. Why Scala library doesn't implement in order dequeue for priority queue?
2. How to implement the Scala priority queue library?


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
