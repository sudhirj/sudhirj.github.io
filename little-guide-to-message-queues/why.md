---
layout: page
title: Why are they useful?
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
        content: Arguing Semantics
        url: /little-guide-to-message-queues/semantics
---

Remember the simple way we thought about the message queue: a tube into which we rolled a ball with our message on it, expecting someone or something else to pick it up from the other end. What advantages do we see from the this model, when applied to designing software systems?

* We don't need to worry about *who or what* is going to receive the message – that's one less responsibility for the code that sends the message. 
* We don't need to worry about *when* the receiver is going to pick up the message.
* We can put *as many* messages as we want into the tube (let's assume we have a very long tube) at whatever *rate* is comfortable to us. 
* The receiver will *never be affected* by our actions - they will pull out as many messages as they want at whatever rate is comfortable to them.
* Neither the sender nor the receiver are concerned with *how* the other works. 
* Neither the sender nor the receiver are concerned with the *capacity or load* of the other.
* Neither system is concerned with *where* the other one is – they might or might not reside on the same computer, network, continent or even the same planet.

Each of these advantages (and this isn't even an exhaustive list) has very important benefits in software development – what they all have in common is *decoupling*. One system is decoupled from the other in terms of responsibility, time, bandwidth, internal workings, load and geography. And decoupling is a very desirable part of any distributed or complex system – the more decoupled the parts of the system are, the easier it is to independently build, test, run, maintain and scale them.

Most systems interact with other outside systems as well – if we build a shopping site we might interact with a payment processor, and let’s say we attempt to directly communicate with the payment processor on each user click. If our system is under heavy load, we're also subjecting the other system to the same load. And vice versa – if our payment provider needs to send us millions of pieces of information about our past payments, our system better be ready. The two systems are now *coupled*. The decisions and actions made by one system have a significant impact on the other, so the needs of both need to be taken into account while making every decision. Add enough other systems into the mix, like logistics or delivery systems, and we quickly have a paralysing mess that makes it difficult to decide anything at all. If one system goes down, the other systems have effectively gone down as well, for no fault of their own. 

We’re also in trouble if we want to switch out any one of these systems for another one, like a new payment processor or delivery system. We’d have to make deep changes in multiple places in our application, and it’s even more difficult to build code to split our messages between multiple providers – we may want to use a ratio or split them by geography; or dynamically switch between them based on each provider’s availability or cost. 

Message queues offer the decoupling that solves a lot of these problems. If we set up a queue between two systems that need to communicate with each other, they can now go about their work without having to consider each other at all – we put our messages aimed at any system into a queue, and we expect information from the other system to come to us through a queue as well. We now have clear points at which we can add rules or make the changes we require, without either system knowing or caring about what's different.

### So what's the catch?

Are message queues the holy grail of computing, though? Do they solve all the world's problems? No, of course not. There are plenty of situations where we might not want to use them. And we certainly don't want to use a queue just because we have one easily available and think it might be fun. There are some systems that are really simple that just don't require it – a message queue is a way to reduce to complexity of communicating systems, but two communicating systems will always be more complex than one system that doesn't have to communicate. If you have a system that’s simple enough to not require communication with any others, there simply isn't any reason to reach for a queue.

There are also systems that communicate with each other, but where the bandwidth required is insignificant and not worth worrying about. Or more often the systems are already coupled, in the sense that they all need to work together to function. A really common example is an application server and a database (in an OLTP[^oltp] system). There's not much point in decoupling them with a queue, because neither can do anything useful without the direct involvement of the other. 

[^oltp]: In an OLTP or Online Transactional Processing system, we want everything to happen as we ask for it to happen, without any delays. This is usually because we need to perform a sequence of steps where each one depends on the one before. The opposite would be OLAP, or Online Analytical Processing systems, where we usually send in instructions and wait for a response. OLAP systems will often use a message queue.

Then there's performance to consider as well – the whole point of decoupling two systems with regards to time and load is so that they can each process information at their own pace – but we certainly would *not* want this to happen in performance sensitive applications or real-time systems. A queue might help us process more work at the same time (the receiver might have many processes working in parallel on the messages you send) but will remove any guarantees we need about the exact time taken for each piece of work. If predictability is more important than throughput[^throughput], we're better off without a queue.

[^throughput]: Using a queue might make each individual message slower, but will allow you process many more messages at the same time – so your total number of messages processed per minute or hour (throughput) will increase.

But if we do have multiple systems that need to communicate, and that communication needs to be durable[^durable] and may be unpredictable in volume, a message queue is indispensable. 

---

[^durable]: If we’ve put a message into a queue, we want to be sure that the messaging system isn’t going to ‘forget’ about it, under any circumstances. That guarantee is usually called *durability*, as defined in the [ACID](https://en.wikipedia.org/wiki/ACID) list of guarantees.





