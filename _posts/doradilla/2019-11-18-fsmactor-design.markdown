---
layout: post
comments: true
title:  "FSMActor workflow design"
date:   2019-11-18 15:22:16 +0800
categories: jekyll update
img: fsm.jpg # Add image post (optional)
tags: [scala, akka, doradilla, fsm, state machine, resilient design]
---

FSMActor is the engine of the job management. The job work flow as below:

![workflow](/media/fsmactor/workflow.jpg)

1.	The ProxyActor put the request job to QueueActor
2.	When FSMActor in idle status, the FSMActor will retrieve job from QueueActor
3.	QueueActor will send the queued job to FSMActor
4.	FSMActor will tell ProxyActor that the job is started and ProxyActor will record which FSMActor is handle the job, FSMActor will turn to active status. 
5.	FSMActor will send the job to JobTranslator.
6.	JobTranslator translate the job and send JobWorker information and job detail to FSMActor.
7.	FSMActor will create Jobworker and do the job using job detail
8.	When JobWorker finish its job, the job result will send to ProxyActor
9.	When ProxyActor receive the job result,  ProxyActor will send message to FSMActor to ask FSMActor change status to idle to handle new job



FSMActor is a FSM(finite-state machine) Actor which only has two status, Idle and Active as below:

![statemachine1](/media/fsmactor/statemachine1.jpg)

FSMActor start as Idle status, in that status, FSMActor will retrieve a job and change to Active status.

When job finish (Success, Fail, Timeout), the FSMActor will change from Active status to Idle Status.


```scala


  def runProcessCommand(processJob:JobMsg, 
  backendServerOpt: Option[BackendServer] = None, 
  timeout: Timeout = ConstVars.longTimeOut, 
  priority: Option[Int] = None)
  (implicit ex: ExecutionContext): Future[JobResult] = {
    val backendServer = backendServerOpt match {
      case Some(backendServer) => backendServer
      case _ => startup(Some(seedPort))
    }
    val resultOpt= for( driverService<- backendServer.getActorProxy(Const.driverServiceName);
         processTranService<- backendServer.getActorProxy(Const.procssTranServiceName))
      yield{
        val actorSystem = backendServer.actorSystemOpt.get
        val receiveActor = actorSystem.actorOf(ReceiveActor.receiveActorProps, CNaming.timebasedName("Receive"))
        val processJobRequest = JobRequest(processJob, receiveActor, processTranService, priority)
        driverService.tell(processJobRequest, receiveActor)
        val result = (receiveActor ? FetchResult()) (timeout).map {
          result =>
            receiveActor ! ProxyControlMsg(PoisonPill)
            receiveActor ! PoisonPill
            result.asInstanceOf[JobResult]
        }
        result
      }
    resultOpt.getOrElse(Future(JobResult(JobStatus.Failed, new Exception(JsError("Can't get service")))))
  }

```

What’s the issue of the code above? 

```scala
val result = (receiveActor ? FetchResult()) (timeout).map {
    result =>
    receiveActor ! ProxyControlMsg(PoisonPill)
    receiveActor ! PoisonPill
    result.asInstanceOf[JobResult]
}

```

Actually when timeout happened, the code can’t handle the situation. The FSMActor will not reset to Idle status.

Let handle the timeout situation via the code as below:
```scala
def runProcessCommand(processJob: JobMsg,
 backendServerOpt: Option[BackendServer] = None,
  timeout: Timeout = ConstVars.longTimeOut,
   priority: Option[Int] = None)
   (implicit ex: ExecutionContext): Future[JobResult] = {
    val backendServer = getBackendServerForCommand(backendServerOpt)
    val resultOpt = for (driverService <- backendServer.getActorProxy(Const.driverServiceName);
                         processTranService <- backendServer.getActorProxy(Const.procssTranServiceName))
      yield {
        val actorSystem = backendServer.actorSystemOpt.get
        val receiveActor = actorSystem.actorOf(ReceiveActor.receiveActorProps, CNaming.timebasedName("Receive"))
        val processJobRequest = JobRequest(processJob, receiveActor, processTranService, priority)
        driverService.tell(processJobRequest, receiveActor)
        implicit val timeoutValue: Timeout = timeout
        var result = JobResult(JobStatus.Unknown, "Unkonwn").asInstanceOf[Any]
        try {
          result = Await.result((receiveActor ? FetchResult()), timeout.duration)
        } catch {
          case ex: Throwable =>
            Logger.apply(this.getClass.getName).error(s"$processJob timeout after $timeoutValue")
            result = JobResult(JobStatus.TimeOut, ex.toString)
        }
        receiveActor ! ProxyControlMsg(PoisonPill)
        receiveActor ! PoisonPill
        Future(result.asInstanceOf[JobResult])

      }
    resultOpt.getOrElse(Future(JobResult(JobStatus.Failed, new Exception(JsError("Can't get service")))))
  }
```

The timeout situation will be handled, but the code changes the operation to blocked operation with Await. So we need to wrap the blocked IO with Future.

``` scala
  def runProcessCommand(processJob: JobMsg,
   backendServerOpt: Option[BackendServer] = None,
    timeout: Timeout = ConstVars.longTimeOut, 
    priority: Option[Int] = None)
    (implicit ex: ExecutionContext): Future[JobResult] = {
    val backendServer = getBackendServerForCommand(backendServerOpt)
    val resultOpt = for (driverService <- backendServer.getActorProxy(Const.driverServiceName);
                         processTranService <- backendServer.getActorProxy(Const.procssTranServiceName))
      yield {
        val actorSystem = backendServer.actorSystemOpt.get
        val receiveActor = actorSystem.actorOf(ReceiveActor.receiveActorProps, CNaming.timebasedName("Receive"))
        val processJobRequest = JobRequest(processJob, receiveActor, processTranService, priority)
        driverService.tell(processJobRequest, receiveActor)
        implicit val timeoutValue: Timeout = timeout
        var result = JobResult(JobStatus.Unknown, "Unkonwn").asInstanceOf[Any]
	    def getResult = {
	      implicit val timeoutValue: Timeout = timeout
	      try {
	        result = Await.result((receiveActor ? FetchResult()), timeout.duration)
	      } catch {
	        case ex: Throwable =>
	          Logger.apply(this.getClass.getName).error(s"$jobRequest timeout after $timeout")
	          result = JobResult(JobStatus.TimeOut, ex.toString)
	          receiveActor ! ProxyControlMsg(result)
	      }
	      receiveActor ! ProxyControlMsg(PoisonPill)
	      receiveActor ! PoisonPill
	      result.asInstanceOf[JobResult]
	    }
	    Future(getResult)
      }
    resultOpt.getOrElse(Future(JobResult(JobStatus.Failed, new Exception(JsError("Can't get service")))))
  }
```

In worse case the FSMActor will not reset to timeout, and we could add one configuration which will reset the FSMActor to Idle status. 

![statemachine2](/media/fsmactor/statemachine2.jpg)


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
