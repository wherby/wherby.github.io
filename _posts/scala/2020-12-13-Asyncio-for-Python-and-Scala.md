---
layout: post
comments: true
title: "Asyncio for Python and Scala"
date: 2020-12-13 13:22:16 +0800
categories: jekyll update
img: asyncio.jpg # Add image post (optional)
tags: [asyncio, python, scala]
---

The python part are mainly from [“A Web Crawler With asyncio Coroutines”](https://www.aosabook.org/en/500L/a-web-crawler-with-asyncio-coroutines.html)

Let begin with simple web crawler task, the task is to download urls from website. As we known, the download part is a IO-bound operation.

# Python asyncio

## Start with block-io

```python
def fetch(url):
    sock = socket.socket()
    sock.connect(('xkcd.com', 80))
    request = 'GET {} HTTP/1.0\r\nHost: xkcd.com\r\n\r\n'.format(url)
    sock.send(request.encode('ascii'))
    response = b''
    chunk = sock.recv(4096)
    while chunk:
        response += chunk
        chunk = sock.recv(4096)

    # Page is now downloaded.
    links = parse_links(response)
    q.add(links)
```

There block-io is the connect, send and recv for socket.

What's will happen if use multipe-thread or multipe-process for fetch operation?

Multipe-thread will have the same issue.

Multipe-process will create resource overhead and partially resolve the issue.

## Use non-blocking socket

```python
sock = socket.socket()
sock.setblocking(False)
try:
    sock.connect(('xkcd.com', 80))
except BlockingIOError:
    pass
request = 'GET {} HTTP/1.0\r\nHost: xkcd.com\r\n\r\n'.format(url)
encoded = request.encode('ascii')

while True:
    try:
        sock.send(encoded)
        break  # Done.
    except OSError as e:
        pass
```

What's will happen if use multipe-thread?

## Use select-like function

```python
selector = DefaultSelector()

sock = socket.socket()
sock.setblocking(False)
try:
    sock.connect(('xkcd.com', 80))
except BlockingIOError:
    pass

def connected():
    selector.unregister(sock.fileno())
    print('connected!')

selector.register(sock.fileno(), EVENT_WRITE, connected)


def loop():
    while True:
        events = selector.select()
        for event_key, event_mask in events:
            callback = event_key.data
            callback()
```

There will have some improvement with Future class defined to handle callback.

## Use asyncio lib

```python
@asyncio.coroutine
def fetch(self, url):
    response = yield from self.session.get(url)
    body = yield from response.read()
```

# Scala way

## Scala Future

f1:A=> Future[B]
f2:B=> Future[C]

List[A].map{
A=>f1(A).flatMap(B=>f2(B))
}

## Scala Akka

Event driven

## Doradilla lib

Python use asyncio lib to handle IO-bound task, and yield from in Python is similar to
flatMap in Scala. For python's GIL feature, only one CPU core will be used in one process,
so asyncio can't well handle CPU-bound task without use multipe-process lib.

Scala has two ways(Future and Akka) to handle IO-bound task, and can use multipe-CPU by default, also can handle
CPU-bound task. But when CPU-bound tasks overflows, these two ways will also create more jobs which will introduce
context swith overhead.

Doradilla lib could handle above scenarios easily.
