---
layout: post
comments: true
title:  "Antipattern the secret in Cassandra"
date:   2017-12-18 13:22:16 +0800
categories: jekyll update
---
Cassandra uses the most important and "famous" antipattern design for update value. The update is as below[1]: 

![cassandra](/media/AntiPattern/image1.jpg)

So the following example is the Cassandra DB record of [“Add A”,”Add B”,”Add C”,”Delete A”……]

![cassandra](/media/AntiPattern/image2.jpg)


The antipattern is when delete the record, the Cassandra will not actually delete the record but mark the record as a tombstone, and delete it in latter time.

What’s the benefit of the antipattern: speed and lock free design.

From speed aspect, delete operation is just update the record is much faster than delete the record and rearrange the other records. Just like GC in Java or C#, when you delete an object, what action for the deletion is just mark the object to be delete and wait GC to release the object and rearrange the objects in memory.

From lock free aspect which is more easy to understand, if multi objects are deleted and need to rearrange the records, that must need some lock on the table. And if use mark and delete, the delete operation could focus to do the operation. Just as the following code:

```python
def addls(l,r):
    global ls,validls,dic1
    start = bisect.bisect_left(validls,l)
    end = bisect.bisect_right(validls,r)
    removeIndex = []
    for i in range(start,end):
        tp = validls[i]
        dic1[tp] = dic1[tp] +1
        if dic1[tp] >= 49:
            removeIndex.append(i)
    for i in range(len(removeIndex)):
        k = removeIndex[i]
        validls.pop(k-i) 
```


Suppose the operation in first for loop is operation distributed on various hosts. If update array in one host which will affect the array on other host.

[1] https://www.slideshare.net/doanduyhai/cassandra-nice-use-cases-and-worst-anti-patterns

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
