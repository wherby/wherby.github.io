---
layout: post
comments: true
title:  "Revisit Hydra"
date:   2022-01-02 15:22:16 +0800
categories: jekyll update
img: revisithydra.png # Add image post (optional)
tags: [scala, akka, hydra, roadmap, distributed data, serverless,p2p]
---

3 years ago I finished develop Hydra. So what's problem Hydra resolved?

Let's talk about Akka cluster first.

Akka cluster is a traditional cluster implementation, when cluster setup, there will select a master(leader),
all member's event will use gossip protocol to synchronize. While there may have brain spit or when master is down,
there need time to select new mater first or gossip maybe slow.

In original plan, the Hydra could detected failer in consistent time, which means the failure will be process by any member of
node which has scheduler, and the state of deployment will be recorded in distributed data, which will have eventually consistent status 
with gossip protocol. When the role of deploy operation changed from master node to peer node, after cluster setup, the status can be managed by
peer-to-peer node, then master don't have much task to handle, event the master is down, the HA behavior may not be impacted during master re-selection.

Why I don't develop Hydra as original plan, the truth is the right way use akka maybe is the original plan, which means keep the cluster feature 
as little as possible and develop less status related to cluster data. We can use single truth principle to re-architecture application at very beginning stage. 

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
