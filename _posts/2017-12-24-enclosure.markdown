---
layout: post
comments: true
title:  "Enclosure for Akka Props"
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

What’s the definition tells is that, the Props will add construction to same type. In another word, there will create some class declaration in the environment. Just as Python Case 2, the inner class will enclose the outer class. When the inner actor deployed in remote, not only the inner class will be serialized, but the outer class reference without conscious. For more detail, see: 
[https://www.cakesolutions.net/teamblogs/understanding-akkas-recommended-practice-for-actor-creation-in-scala](https://www.cakesolutions.net/teamblogs/understanding-akkas-recommended-practice-for-actor-creation-in-scala)

Define a class in other class may be OK and as a syntax in other language like C#, Java. But if the code used in distributed environment, there may be more consideration for the syntax.


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
