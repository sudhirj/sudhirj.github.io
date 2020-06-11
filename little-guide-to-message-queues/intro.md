---
layout: page
title: Introduction
author:
    title: The Little Guide to Message Queues
    title_url: /little-guide-to-message-queues/
    external_url: false
    description: A short guide to what, why and wtf! of message queues.
page_nav:
    prev:
        content: Index
        url: /little-guide-to-message-queues/
    next:
        content: What are they?
        url: /little-guide-to-message-queues/what
---

Message Queues are now fairly prevalent – there are more of them showing up so fast you'd think they were rabbits[^1] with an unlimited supply of food[^2], resulting in an Kafkaesque[^3] situation where making a decision is like trying to catch a stream[^4] in your hands. If only there were fewer simple services[^5] that could help with publishing and subscribing[^6], it would be so much easier to make a zero-effort[^7] choice. 

But either used by themselves as important building blocks of complex applications; or as an integral part of patterns like event driven architectures, message queues are here to stay. In a way, they've been here all along – just without this many names. But what are they? Why are they useful? And how do we use them effectively? Which one do we pick? Does it even matter which one we use? And do we need to learn each of them individually, or are there more general concepts that apply to all message queues?

We'll talk about these questions, and more, in this booklet – *The Little Guide to Message Queues*. Our main focus will be on the serverless offerings on Amazon Web Services (AWS) – the Simple Notification Service (SNS) and Simple Queue Service (SQS). Both are fully managed systems that are hosted and run by AWS, so customers can use them at whatever capacity they need and pay per message or API request. We're looking at these tools because they've stood the test of time and work effectively at massive scales; while still being cost-effective (really cheap), simple to understand, and easy to use. 

The concepts we'll discuss about are universal, though, and most self hosted messaging systems and those on any cloud infrastructure provider will offer similar APIs and ways of using them. We'll also see how to apply what we've learned on Apache Kafka, Redis Streams and RabbitMQ[^8].

In this guide, we'll talk about:

* What message queues are and their history. 
* Why they're useful and what mental models to use when reasoning about them.
* Delivery guarantees that the systems make (at-least-once, at-most-once, and exactly-once semantics).
* Ordering and FIFO guarantees and how they effect parallelism and performance.
* Patterns for fan-out and fan-in: delivering one message to many systems or messages from many systems into one.
* Specific notes on how to best use the SQS (Simple Queue Service) and SNS (Simple Notification Service) systems on AWS, explaining the important configuration options on each API call and common mistakes to watch out for.
* Notes for running on a few other systems as well, like Kafka, Redis and RabbitMQ.

This is a short booklet, and not meant to be a comprehensive guide to anything – my aim is more to introduce you to ideas and concepts that I've found helpful, and provide links to more information as much as possible. The book is still a work in progress, so do get in touch at [sudhir.j@gmail.com][11] or [@sudhirj][12] if there's anything you think should be included or changed.

---

[^1]:	[RabbitMQ][1]

[^2]:	[Celery][2]

[^3]:	[Apache Kafka][3]

[^4]:	[Redis Streams][4]

[^5]:	AWS [SNS][5] & [SQS][6]

[^6]:	[Google Pub/Sub][7]

[^7]:	[ZeroMQ][8]

[^8]:	These are the ones I think might be useful to write about so far, but if you'd like notes on any other systems get in touch with me at [sudhir.j@gmail.com][9] or [@sudhirj][10]

[1]:	https://www.rabbitmq.com/
[2]:	http://www.celeryproject.org/
[3]:	https://kafka.apache.org/
[4]:	https://redis.io/topics/streams-intro
[5]:	https://aws.amazon.com/sns/
[6]:	https://aws.amazon.com/sqs/
[7]:	https://cloud.google.com/pubsub
[8]:	https://zeromq.org/
[9]:	mailto:sudhir.j@gmail.com
[10]:	https://twitter.com/sudhirj
[11]:	mailto:sudhir.j@gmail.com
[12]:	https://twitter.com/sudhirj
[13]:	https://gumroad.com/l/little-guide-to-message-queues