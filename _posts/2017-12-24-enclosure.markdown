---
layout: post
comments: true
title:  "Enclosure for Akka's Props"
date:   2017-12-24 13:22:16 +0800
categories: jekyll update
img: enclosure.jpg # Add image post (optional)
tags: [scala, python, enclosure, akka]
---

Let's begin with 3 python examples:

```python
[Case 1]
class class2:
    def printS(self):
        print "Inside Class2"
def fun1():
    c2 = class2()
    c2.printS()

fun1()

#Inside Class2
#[Finished in 0.2s]

```

```python
[Case 2]
class class1:
    class class2:
        def printS(self):
            print "Inside Class2"
    def fun1(self):
        c2 = class2()
        c2.printS()

c1 = class1()
c1.fun1()

#  File "C:\test2.py", line 10, in <module>
#    c1.fun1()
#  File "C:\test2.py", line 6, in fun1
#    c2 = class2()
#NameError: global name 'class2' is not defined

```

```python
[Case 3]
class class1:
    class class2:
        def printS(self):
            print "Inside Class2"
    def fun1(self):
        c2 = self.class2()
        c2.printS()

c1 = class1()
c1.fun1()

#Inside Class2
#[Finished in 0.1s]

```

The three examples are quiet simple cases:

    in case 1, everything goes well

    in case 2, when define class in another class, the directly call the class will throw exception

    in case 3, when call self.class2, it works again.

The lesson from the case 2 is that, when define a class inside of another class, the inner one is not dependence class. The behavior is not a big deal in traditional programs, but will trigger a disaster in distributed system.

When start with Akka, you will meet the alert like this:
[https://doc.akka.io/docs/akka/current/actors.html?language=scala](https://doc.akka.io/docs/akka/current/actors.html?language=scala)

```scala
// NOT RECOMMENDED within another actor:
// encourages to close over enclosing class
val props7 = Props(new MyActor)

```

But what’s the relation of the two different things? Are Props and class declaration different?
When look up Props definition:

```scala
/**
 * Scala API: Returns a Props that has default values except for "creator" which will be a function that creates an instance
 * of the supplied type using the default constructor.
 */
def apply[T <: Actor: ClassTag](): Props = apply(defaultDeploy, implicitly[ClassTag[T]].runtimeClass, List.empty)

```

What’s the definition tells is that, the Props will add construction for some type. In another word, there will create some class declaration in the environment. Just as Python Case 2, the outer class will be enclosued in the inner class. When the inner actor is deployed to remote, not only the inner class will be serialized, but the outer class reference without conscious. For more detail, see: 
[https://www.cakesolutions.net/teamblogs/understanding-akkas-recommended-practice-for-actor-creation-in-scala](https://www.cakesolutions.net/teamblogs/understanding-akkas-recommended-practice-for-actor-creation-in-scala)

Define a class in other class may be OK and as a syntax in other language like C#, Java. But if the code used in distributed environment, there may be more consideration for the syntax.


# Advanced topic

## name hidden effect

If we have a function to run processor in dynamic way using reflection feature as blow:

``` Scala
object DoraServer {
  var  backendServerOpt: Option[BackendServer] = None
…

  def runProcess(paras : List[AnyRef], clazzName:String, methodName:String, prioritySet: Option[Int] = None) ={
    val setTimeOut:Timeout = 6000 seconds
    val msg = ProcessCallMsg(clazzName,methodName,paras.map{_.asInstanceOf[AnyRef]}.toArray)
    val jobMsg = JobMsg("SimpleProcessFuture",msg)
    BackendServer.runProcessCommand(jobMsg,priority = prioritySet,timeout = setTimeOut).map{
      jobResult=>
        println(jobResult)
        if((jobResult.result.asInstanceOf[ProcessResult]).jobStatus.toString == "Failed" ){
          println("failed....")
          throw new RuntimeException(jobResult.result.asInstanceOf[ProcessResult].result.asInstanceOf[Exception].getMessage)
        }
        jobResult.result.asInstanceOf[ProcessResult].result.asInstanceOf[ProcessorResultValue]
    }
  }
}
```

But you will have the processor which is defined in another class as below:
``` Scala 
class MockProcessorSpec extends AsyncFlatSpec with BeforeAndAfterEach with SequentialNestedSuiteExecution{


  class MockProcessor() extends ConcurrentProcessor() {
    override def propagate(branch:Map[String, ProcessorBranch], context:ContextValue):Map[String, SubJobConfig] = {
....
    }

}

```

When you want to run the MockProcessor with the dynamic way, then you will find you can’t initialize the MockProcessor, because the MockProcessor class only exists in the MockProcessorSpec instance scope, which created a “namespace hidden effect”.

If you want to call MockProcessor in dynamic way, you need to reorganize the class as below:
But you will have the processor which is defined in another class as below, expose the class in global scope:
``` Scala 
class MockProcessorSpec extends AsyncFlatSpec with BeforeAndAfterEach with SequentialNestedSuiteExecution{



}

class MockProcessor() extends ConcurrentProcessor() {
    override def propagate(branch:Map[String, ProcessorBranch], context:ContextValue):Map[String, SubJobConfig] = {
....
    }


```

## Name hidden effect in design pattern

So you could see the name hidden effect when class defined in another class. So how could the effect be used in reality

For example, Slick will auto generate some classes which are map to database implementation and these classes don’t need to be exposed to business code.

Slick auto-generated code:
``` Scala
trait Tables {
....


  case class CountryCityRow(id: Int, country: String, province: Option[String] = None, city: String)

...
}
```
And you will implement the class extend “Tables” which will use the “CountryCityRow” which is defined in the “Tables” class. In this way, only SlickCityDAO which extends the “Tables” class could access “CountryCityRow”.
``` Scala 

@Singleton
class SlickCityDAO @Inject()(db: Database)(implicit ec: ExecutionContext) extends CityDAO with Tables {

  import profile.api._

...

  private def convertObjectToRow(data: CountryCityFront): CountryCityRow = {
    CountryCityRow(data.id, data.country, data.province, data.city)
  }

  private def convertRowToObject(row: CountryCityRow): CountryCityFront = {
    CountryCityFront(row.id, row.country, row.province, row.city)
  }


}

```


For “CountryCityRow” is the implementation which autogenerated by Slick, if use the “name hidden effect”, only the class needs the class could access that class.


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
