---
layout: post
comments: true
title:  "The Tao in database [To be continue…]"
date:   2018-1-23 13:22:16 +0800
categories: jekyll update
img: taoofdatabase.png # Add image post (optional)
tags: [tao, cassandra,  streaming, database, timetravel]
---

Let’s begin with some existed databases:
## X-DB
The X-DB in TaoBao in Alibaba, the following is from TaoBao’s blog, which could handle 325K OPS (Operations Per Second) [ http://jm.taobao.org/2017/12/27/20172701/](http://jm.taobao.org/2017/12/27/20172701/):

![Tao bao](/media/TaoOfDatabase/Taobao.png)

X-DB is a SQL database in the biggest e-commerce company in china, so the cluster size and maintenance effect is not quiet realistic for small business. But how can we achieve the same performance database with little cost? Use No-SQL database!!!

## Cassandra

Netflix has a benchmark of Cassandra [https://medium.com/netflix-techblog/benchmarking-cassandra-scalability-on-aws-over-a-million-writes-per-second-39f45f066c9e](https://medium.com/netflix-techblog/benchmarking-cassandra-scalability-on-aws-over-a-million-writes-per-second-39f45f066c9e):

![Cassandra](/media/TaoOfDatabase/Netflix.png)

To achieve the same IOPS, only need around 40 m1.xlarge EC2 instance. Which is more acceptable (less than 30$ / hour).

## Scylla


So can it be more quick? Use Scylla, see the benchmark of the new database which rewrite Cassandra [http://www.scylladb.com/product/benchmarks/](http://www.scylladb.com/product/benchmarks/)

![Scylla](/media/TaoOfDatabase/scylla.png)

Scylla only use 10% nodes to achieve same performance of Cassandra clusters.


## FiloDB

But what’s more powerful database is based on Stream job of No-SQL.

The following architecture is from [Evan Chan](https://www.linkedin.com/in/evanfchan) 's FiloDB [FiloDB](https://www.slideshare.net/EvanChan2/2017-high-performance-database-with-scala-akka-spark):

![Filo1](/media/TaoOfDatabase/Filo1.png)

The benchmark as below [https://www.slideshare.net/EvanChan2/2017-high-performance-database-with-scala-akka-spark](https://www.slideshare.net/EvanChan2/2017-high-performance-database-with-scala-akka-spark):

![Filo2](/media/TaoOfDatabase/Filo2.png)

That FiloDB will achieve **billion level OPS** with **“single thread”**!!

# The above databases tell:

1.  To achieve high OPS, use No-SQL

2.  Using stream to powerup No-SQL will achieve no-limited performance [Similar idea in the tao of web service [https://wherby.github.io/the-tao-in-web-service/](https://wherby.github.io/the-tao-in-web-service/)]

# Tao in database [To be continue…]

Is there any defect in present database? What’s the future of database?

Yes, present database (both SQL or No-SQL) is just “real time snapshot” of real world, which hidden the history of data.

The future of database may have the “time travel” ability.

[To be continue…]



[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
