---
layout: post
comments: true
title:  "Introduction to Doradilla"
date:   2019-12-10 15:22:16 +0800
categories: jekyll update
img: doradilla.jpg # Add image post (optional)
tags: [scala, akka, doradilla, job manage, akka cluster]
---

# What’s Doradilla

Doradilla is a job manage library which provides reactive way to handle job request.


# Doradilla introduction

## What's the problem the library resolve?

The library provides a reactive way to handle resource consuming(CPU, Memory, DB connection) tasks.

For example, an OCR application which will trigger OCR tasks based on requests, for each OCR task there needs one CPU core occupied. If there is no implementation of job management, the CPUs will be easily taken by OCR jobs. The CPU competition will easily slow down the processing and block other functions.

#### What's the traditional way to solve the issue?

create a job queue, and use workers to take job from the queue.

#### Is there any universal way to resolve this type of question and makes the implementation easy to use?

Yes, just use the Doradilla library.

## How the Doradilla library works?

#### Simple version:

The Doradilla library use a queue to keep job requests and FSMActor will pull job request to process.

#### Is the same way as traditional way?

Yes, but not, because the user will not aware of the library implementation. The example shows user only call the job api, the Doradilla library will handle the travail work.

## Why there is need JobTranslator?

For general purpose, every complex job could be translate to simple jobs and so on. When you handle the complex job, you could design your JobTranslator to handle that job.

## What's if I aready have ActorSystem in my project?

Well, you can only use doradcore library instead of use doradilla.



# Message flow

![message flow](/media/doradilla/msgflow.jpg)


# Doradilla cluster usage

Doradilla provides distributed running environment which is based on Akka cluster. With same configuration as Akka cluster, Doradilla-core will running on Akka cluster node.

![dora cluster](/media/doradilla/dora-cluster.png)


# How to use

## How to run a process job
#### Run process job and get synchronize result:
```scala
"Baeckend server " should "start and run command " in {
  val backendServer = BackendServer.startup(Some(1600))
  backendServer.registFSMActor()
  val msg = TestVars.processCallMsgTest
  val processJob = JobMsg("SimpleProcess", msg)
  val res = BackendServer.runProcessCommand(processJob).map {
    res =>
      println(res)
      assert(true)
  }
  Await.ready(res, ConstVars.timeout1S * 10)
}
```

#### Run process job and query result:

``` scala
"Run process Command " should  "start the command and qurey result " in {
  val backendServer = BackendServer.startup(Some(1600))
  backendServer.registFSMActor()
  val msg = TestVars.processCallMsgTest
  val processJob = JobMsg("SimpleProcess", msg)
  val receiveActor = BackendServer.startProcessCommand(processJob).get
  BackendServer.queryProcessResult(receiveActor).map {
    resultOpt =>
      assert(resultOpt == None)
  }
  Thread.sleep(2000)
  val res = BackendServer.queryProcessResult(receiveActor).map {
    resultOpt =>
      assert(resultOpt != None)
  }
  Await.ready(res, ConstVars.timeout1S)
}
```

## For use defined implementation of reflection.
User should defined their implementation of reflection, more information see [ProcessService](https://github.com/wherby/doradilla/tree/master/docs/doradilla-core/util/ProcessService.md)

## More link about this library

[FSMActor workflow design](https://wherby.github.io/fsmactor-design/)

## More information

[Doradilla github](https://github.com/wherby/doradilla)

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
