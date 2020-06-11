---
layout: page
title: Arguing Semantics
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
        content: Ordering vs Parallelism
        url: /little-guide-to-message-queues/ordering-parallelism/
---

There's simply no way to learn about message queues without reading or/and arguing about delivery guarantees and semantics, so we might as well get to that quickly. People who build message queues will claim that their system offers one of three delivery guarantees – that each message you put into the queue will be delivered:

* at least once.
* at most once.
* exactly once.

Which guarantees we're using will have a massive impact on the design and working of our system, so let's unpack each of them one by one. 

## At Least Once

This is the most common delivery mechanism, and it's the simplest to reason about and implement. If I have a message for you, I will read it to you, and keep doing so again and again until you acknowledge it. That's it. In AWS SQS, which works on an at-least-once basis, this means that when you receive a message from the queue and don't delete it, you will receive it again in the future, and will keep receiving it until you explicitly delete[^1] it.

The reason this is the most common guarantee is that it's simple and gets the job done 100% of the time – there's no edge case in which the message gets lost. Even if the receiver crashes before acknowledging the message, it will simply receive the same message again. The flip side is that you as the receiver need to plan on receiving the same message multiple times – even if you haven't necessarily experienced a crash. This is because offering at-least-once is the simplest way to protect the queueing service from missing out messages as well – if your acknowledgement doesn't reach the queueing system over the network, the message will be sent again. If there's a problem persisting your acknowledgement, the message will be sent again. If the queuing system restarts before it can properly keep track of what's been sent to you, the message will be sent again. This simple remedy of sending the message again in case of any problem on any side is what makes this guarantee so reliable. 

But is message duplication a problem? That's really up to you and your application / use-case. If the message is a timestamp and a measurement, for example, there's no problem with receiving a million duplicates. But if you're moving money based on the messages, it definitely is a problem. In these cases you'll need to have a transactional (ACID) database at the receiving end, and maybe record the message ID in a unique index so that it can't be repeated. This is called using an *idempotency token*[^2] or *tombstone*[^3] – when you act on a message you store a unique permanent marker to keep track of your action(s), often in the same database transaction as taking the action itself. The prevents you from repeating that action again even if the message is duplicated. 

If you handle duplication, or if your messages are naturally resistant to duplication, your systems are said to be ‘idempotent’. This means you can safely handle receiving the same message multiple times, without corrupting your work. It also often means you can tolerate the sender *sending* the same message multiple times – remember that senders will usually operate on the at-least-once principle when sending messages as well. If senders are unable to record the fact that they've sent a particular message, they'll simply send it again. The senders are then responsible for making sure that they use the same tombstone or idempotency token if and when they re-send messages.

## At Most Once

This is a pretty rare semantic, used for messages where duplication is so horribly explosive (or the message so utterly unimportant) that we'd prefer *not* to send the message at all, rather than send it twice. At-most-once once implies that the queuing system will attempt to deliver the message to you once, but that's it. If you receive and acknowledge the message all is well, but if you don't, or anything goes wrong, that message will be lost forever – either because the queuing system has taken great pains to record the delivery to you *before* attempting to send it (in case the message is horribly explosive), or has not even bothered to record the message at all, just passing it on like a router passes on a UDP packet[^4].

This semantic usually comes into play for messaging systems that are either acting as stateless information routers; or in those cases where a repeat message is so destructive that an investigation or reconciliation is necessary in case there's any failure.

## Exactly Once

This is the holy grail of messaging, and also the fountain of a lot of snake-oil. It implies that every message is guaranteed to be delivered and processed exactly once, no more and no less. Everyone who builds or uses distributed systems has a point in their lives where they think “how hard can this be?”, and then they either (1) learn why it's impossible, figure out idempotency, and use at-least-once, or (2) they try to build a half-assed “exactly-once” system and sell it for lots of money to those who haven't figured out (1) yet. 

The impossibility of exactly once delivery arises from two basic facts:
 
 1. senders and receivers are imperfect
 2. networks are imperfect

If you think about the problem deeply, there are a lot of things that can go wrong:

* a sender might be unable to record (’forget’) that they've sent the message
* the network call to send the message might fail
* the messaging system’s database might not be able to record the message
* the acknowledgement that the messaging system has recorded the message might not reach the sender over the network
* the sender might not be able to record the acknowledgement that the messaging system has received the message

Let's say all goes well while sending the message – when the messaging system tries to deliver the message to the receiver:

* the message might not reach the receiver over the network
* the receiver might not be able to record the message in its database
* the acknowledgement from the receiver might not reach the messaging system over the network
* the messaging system’s database might not be able to record that the message has been delivered

Given all the things that can go wrong, it's impossible for any messaging system to guarantee exactly-once delivery. Even if the messaging system is godlike in its perfection, most of the things that can go wrong are outside of it or in the interconnecting networks. Some systems do attempt to use the phrase “exactly once” anyway, usually because they claim their implementation will never have any of the messaging system problems mentioned above – but that doesn't mean the whole system is magically blessed with exactly-once semantics, even if the claims are actually true. 

Most good messaging system engineers understand this and will explain to their users[^5] why this semantic is unworkable. The simpler and more reliable way to handle messages is go back to the basics and embrace at-least-once with idempotency measures at every point on the sending, receiving and queuing process: if at first you don't succeed, retry, retry, retry... 

---

[^1]:	*Delete* and *Acknowledge* are the two common terms used to inform the message queue system that the message has been processed successfully. SQS uses *delete*, while Kafka and others use *acknowledge*. They both mean the same thing in this context.

[^2]:	Stripe has excellent support for idempotency in all their API calls, check out the notes [here][1].

[^3]:	[Wikipedia][2] talks about the *tombstone* from a database consistency point of view, but I learnt of it’s usage in message queue from Google App Engine’s Task Queue service many years ago – but they seem to have shut it down (surprise!) since.

[^4]:	https://stackoverflow.com/questions/15629329/is-it-possible-to-guarantee-delivery-of-messages-using-udp-on-node-js 

[^5]:	https://www.lightbend.com/blog/how-akka-works-exactly-once-message-delivery

[1]:	https://stripe.com/docs/api/idempotent_requests
[2]:	https://en.wikipedia.org/wiki/Tombstone_(data_store)