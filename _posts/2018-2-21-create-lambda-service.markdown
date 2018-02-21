---
layout: post
comments: true
title:  "Create your own lambda service -- introduction to Hydra 0.2.1"
date:   2018-2-21 13:22:16 +0800
categories: jekyll update
img: lambda.jpg # Add image post (optional)
tags: [hydra, lambda service, alpakka]
---

As we know, Amazon provides Lambda service on AWS, how can we create the Lambda service on ourselves? 

### Let’s start with Hydra 0.2.1 new feature “External Actor Loader”:

External Actor Loader is a loader which will load actor to Hydra in dynamic way. The loader will load the actor defined in jar file which is provided by user. The Actor will be placed under "/user/externalLoader" path of actor system. and the actor uses a separated dispatcher("external-dispatcher").

### What’s the relationship of External Actor Loader and Lambda service?

The answer is a little tricky, the actor to be loaded is the service in lambda service. Actor is event driven model which is same as Lambda service, so when you implement the same logic as in the Lambda service when the Actor will “act as” the lambda service. Meanwhile the Hydra is based on Akka Cluster which will provide native event bus for the actors. Alpakka(https://github.com/akka/alpakka) will provide various ways of Akka stream connectors. 

### Let’s have a quick guide of External Actor Loader:

#### Create an actor:
 ![Create ](/media/LambdaService/create.png)

User only need to provide the jar address and class name for the actor to be load. 

#### Query actors:
 
  ![Create ](/media/LambdaService/query.png)

#### Delete actor:

  ![Create ](/media/LambdaService/delete.png)

Of course, the actor is granted the Hydra’s HA feature, if the node of actor failed, the actor will be redeployed on another node of cluster.


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
