---
layout: post
comments: true
title:  "Hydra release 0.2.0"
date:   2017-12-29 13:22:16 +0800
categories: jekyll update
img: hydra2.jpg # Add image post (optional)
tags: [scala, akka, hydra, HA container, distribute]
---

# So what’s Hydra?

    1.  Hydra is an Akka Cluster based system which provides high available container service for applications.

    2.  Hydra encapsulate Akka Cluster, with zero-configuration, the Hydra cluster system is setup

    3.  Hydra provide HA container based on heterogeneous platforms (both windows and linux)

# Where can I get Hydra code?

    Github : [https://github.com/wherby/Hydra](https://github.com/wherby/Hydra)

# Demo for Hydra:

The following demo will use materials in Hydra 0.2.0 release: 

[https://github.com/wherby/HydraRelease/tree/master/0.2.0](https://github.com/wherby/HydraRelease/tree/master/0.2.0)

Suppose there are two hosts [192.168.1.4, 192.168.1.25], one is windows and another is linux.

1.download the release files to each host, and change the hydra.conf file according to IP address

```
    [hydra.conf on 192.168.1.25]
    akka.remote.netty.tcp.hostname = "192.168.1.25" 
    akka.cluster.seed-nodes=["akka.tcp://ClusterSystem@192.168.1.4:2551"]
```

```
    [hydra.conf on 192.168.1.4]
    akka.remote.netty.tcp.hostname = "192.168.1.4"
    akka.cluster.seed-nodes=["akka.tcp://ClusterSystem@192.168.1.4:2551"]
```

2.Then run “java -jar Hydra.jar” on each host to start Hydra cluster. You are already started the Hydra cluster as below:

    ![startHydra](/media/HydraRelease2/start.jpg)

3.Post 

```
 {
    "appname": "appTest",
    "startcmd":["python demo/app.py"],
    "prestartcmd":["dir"],
    "healthcheck":"http://localhost:5000/health"
 } 
```
to http://192.168.1.25:9000/app to registe the application of Python.

![register](/media/HydraRelease2/register.jpg)

4.Then query where the application is deployed by get of http://192.168.1.25:9000/app/appTest 

![query](/media/HydraRelease2/query.jpg)

5.Get the application status, from step 4, the ip address of the application is return

![health](/media/HydraRelease2/health.jpg)

6.Crash the application, then Hydra will detect the crash and redeploy the application to another host

![crash](/media/HydraRelease2/crash.jpg)

7.Query the new status of application

![requery](/media/HydraRelease2/requery.jpg)

8.Then get the application status 

![reget](/media/HydraRelease2/reget.jpg)


The demo is just for detection of the application fail, if you crash the host for the application, the Hydra will also handle.

And applications other than Python are supported by Hydra.  

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
