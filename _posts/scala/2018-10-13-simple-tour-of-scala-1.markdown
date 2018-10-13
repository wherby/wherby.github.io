---
layout: post
comments: true
title:  "Simple tour of Scala -- monad"
date:   2018-10-13 13:22:16 +0800
categories: jekyll update
img: monad.jpg # Add image post (optional)
tags: [scala, monad, option]
---


Different from Go which is created by IT giant Google, Scala is created by Martin Odersky in EPFL.  

[A Brief, Incomplete, and Mostly Wrong History of Programming Languages]( http://james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.html)
``` 
2003 - A drunken Martin Odersky sees a Reese's Peanut Butter Cup ad featuring somebody's peanut 
butter getting on somebody else's chocolate and has an idea. He creates Scala, a language that 
unifies constructs from both object oriented and functional languages. This pisses off both groups 
and each promptly declares jihad.
```

## Why Scala?

Scala ecosystem could provide fast data processing toolkit.

![scala cases](/media/monad/scala.png) 

[case study]( https://www.lightbend.com/case-studies)


## What’s Scala?
1.  Scala is object oriented language with concise syntax
2.  Scala is a functional language which is based on monad

## What’s monad?

```
A monad is just a monoid in the category of endofunctors, what's the problem?

一个单子（Monad）说白了不过就是自函子范畴上的一个幺半群而已，有什么难以理解的。

            --from <<Categories for the Working Mathematician>>
```

 ![monad](/media/monad/monad.png) 

[monad ]( https://www.jianshu.com/p/31377066bf97)

[编程语言伪简史]( https://www.oschina.net/news/41233/brief-incomplete-and-mostly-wrong/)


## What’s monad’s meaning, in human’s language?

Take monad factor Option for example:

Suppose you need finish a function of sum two input integer:
```python

def addInt(input):
    if "a" in input and "b" in input:
        a = input["a"]
        b = input["b"]
        try:
            int(a)
            try:
                int(b)
                return a+b
            except:
                print b + " is not int"
        except:
            print a + " is not int"
    else:
        print "a or b not in input parameters"
```

The function will have two types of outputs:
1.  If the inputs are two integers, then return the sum
2.  Else return nothing

For a simple function which may be Ok to have two types of output, if a system has a chain of hundreds function, the output types will be enormous, then the system will be impossible to test and manage.

How to fix the issue?
Scala introduces the Option monad factor:

```scala
def addInt(requst:String):Option[Int]={
  val inputParas = Json.parse(requst)
  val a = (inputParas   \ "a").asOpt[Int]
  val b = (inputParas \ "b").asOpt[Int]
  (a,b) match {
    case (a1:Some[Int],b1:Some[Int])=>Some(a1.get +b1.get)
    case _=>None
  }
} 
```

The Option factor combines the two results:
1.  If in good way, there will return Some[Int]
2.  If in bad way, there will return None

So there will have only one result type, then the system will have limited status. But, wait, the Some and None are still different types. Well, see the code as below:
 
![option](/media/monad/option.png) 
 

The Option factor will combine Some[T] and None to Option[T] which could be used as both input and output, then the chain of functions will work in monad way:

## What's monad in picture?

[Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)
 
![monadway](/media/monad/monadway.png) 

## What're the monadic factors?


Option,Either, Future,Seq,and etc.

 

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
