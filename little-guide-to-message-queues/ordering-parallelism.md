---
layout: page
title: Ordering vs Parallelism
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
        content: Fan-Out & Fan-In
        url: /little-guide-to-message-queues/fan-out-fan-in/
---

After delivery semantics, another common question on peoples’ minds is “why can’t we just process messages in parallel while also making sure we process them in order?”. Unfortunately this is another tradeoff imposed on us by the tyranny of logic. Doing work in a sequence and doing multiple pieces of work at the same time are always at conflict with each other. Most message queue systems will ask you to pick one – AWS SQS started by prioritising parallelism over strict ordering; but recently introduced a separate FIFO (first in, first out) queuing system as well, which maintains strict sequential ordering. Before making a choice between the two, let’s go over what the difference is and why there needs to be a difference at all.

Returning to our earlier metaphor for a queue – a long tube into which we roll messages written on a ball – we probably imagined the tube to be just a little wider than a single ball. There's really no way the balls could overtake or pass each other inside the tube, so the only way a receiver could get these messages out is one by one, in the order they were put in. This guarantees strict ordering, but places strong limitations on our receiver. There can *only be one* agent[^1] on the receiver side that's processing each message – if there was more than one, there would be no guarantee that the messages were processed in order. Because each agent processes each message independently, they could each finish and start on the next message at any time. If the are two agents – A & B – and Agent A receives the first message and Agent B the second; Agent B could finish processing the second message and start on the third message even before Agent A is finished processing the first message. Though the messages were *received from the queue* strictly in the order that they were put in (FIFO), if there are multiple receiving agents there’s no way so say each one will *be processed* in that order.  

The agents could use a distributed lock[^2] of some kind to co-ordinate with each other, but this is basically the same as having only one agent. The lock would only allow one agent to work at any given time[^3].

One way for the messaging system to guarantee order would be for the tube to refuse to give out the next ball until and unless the last ball that was received has been destroyed (the last message has been deleted / acknowledged). This is what FIFO queues in general will do – they'll provide the next message only after the last one has been acknowledged or deleted – but this means that only one agent can possibly be working at a time, even if there are `N` agents waiting to receive messages from the queue. 

Sometimes, this is exactly what we want. Some operations are easier to control effectively when we only have to deal with a single agent, like enforcing rules on financial transactions;  respecting rate limits[^4]; or generally processing messages whose formats have been designed assuming they would always be processed in order. But a lot of these ‘benefits’ are not really coming from the decision to use FIFO ordering – any scenario where we have `N` receivers that must somehow co-ordinate their work with each other will benefit from the special case of `N = 1`. The key takeaway is that requiring a guaranteed order means we have to processes messages sequentially on only one receiver at a time.


This restriction also places severe pressure on the queuing system, so you'll find that FIFO queues are often more expensive and have less capacity than their parallel counterparts. This is because the same logical limits apply to the internal implementation of queuing system as well – most work needs to be constrained to a single agent or server, and that system needs to be kept reliable. Any effort to add redundancy requires synchronous co-ordination between the master and the backup services in order to maintain the ordering guarantees. In AWS SQS, the FIFO queues are about 2X more expensive than the parallel queues, and are constrained to 300 messages per second when strict FIFO ordering is required. 

So the only way to move forward with a FIFO message queue is to accept that the entire message processing architecture is going to have an intrinsic speed limit. We could use group headings[^5] inside the queue to denote what messages we want strict ordering on – we might say that all messages under the heading “payments” need to be FIFO, and all the messages under “orders” need to be FIFO, but they don't need to be FIFO with respect to each other. This allows some parallelisation inside the queue (like having two tubes instead of one), but we need to remember that the message bandwidth in each group heading will still be limited.

### Parallel != Random

Does that mean that the ordering in parallel queues is completely random? Sometimes, yes, but usually not. In SQS, the analogy is more that instead of having one tube from the sender to receiver, there are multiple tubes. They might also branch or join each other along the way. This doesn't mean that the order of the messages you roll in are intentionally randomised in any way – across a large number of messages you'd still expect that earlier messages are generally received before the later ones. This is more a *best-effort* ordering, where some effort is make to keep the ordering intact, but because it's already logically impossible, it's simply not a big priority for the system. This also allows a messaging system like SQS to scale up to nearly infinite capacity – because if you're rolling in a lot of messages the queueing system can simply add more tubes. And as you can imagine, this will support any number of receivers at the same time, and any number of senders as well. This simplicity is what allows SQS to scale to mind-boggling numbers, including a case where there was a queue with over 250 billion messages waiting to be consumed, with the receiver reading and acknowledging over a million messages a second[^6]. And that’s just one queue operated by one customer.  

Most problems that seem like they have a hard FIFO requirement can often lend themselves to parallelism and out-of-order delivery with a little bit of creativity. The sender adding a timestamp into the message is one way to help with this, like in the case where the messages are measurements where only the last one matters. In a more transactional system, the sender can often add a monotonically increasing[^7] counter into the messages. If that's impossible, we might be able to handle this based on the contents of the message – if we're messaging the percentage of a file downloaded, for example, seeing 41%, 42% and 43% always means that the current value is 43% – even if we see them out of order as 41%, 43% and 42%[^8].

While it's often a bad idea to change our systems to accommodate the tools we use, designing our messages to allow for out-of-order delivery lets us use more parallel messaging systems, often saving time, money and a lot of operational work. 

---

[^1]:	The word *agent* refers to a thread, process, instance or container of the same service. Basically each running copy of the receiver code.

[^2]:	Most popular database systems will offer some form of distributed lock to help coordinate between your servers. Redis has [Redlock][1] and PostgreSQL has [Advisory Locks][2], for instance.

[^3]:	But that doesn't mean it's a bad idea. If your code is prone to crashing, or just generally for the sake of better availability, you might want to do this.

[^4]:	It’s much easier to keep a local counter running in a single process – if we have many instances we’ll have to reach for a distributed rate counting system, like [using the Redis `INCR` command with a time based key][3].

[^5]:	AWS FIFO-SQS calls this the [Message Group ID][4]

[^6]:	253,477,653,099 to be precise. And 1.56 million messages processed per second. From [@timbray][5].

[^7]:	There’s a guide to doing it on Postgres [here][6]. There’s also a [talk by Tim Bray][7] about handling some of these problems in general when doing event driven architectures at Amazon.

[^8]:	If you have a particularly difficult problem handling out-of-order messages and are willing to share it with everybody, do get in touch at [sudhir.j@gmail.com][8] or [@sudhirj][9]. We can try working out and solution and publishing it here, or we can mention it as a counter-example if it's impossible.

[1]:	https://redis.io/topics/distlock
[2]:	https://www.postgresql.org/docs/12/explicit-locking.html#ADVISORY-LOCKS
[3]:	https://redis.io/commands/incr#pattern-rate-limiter
[4]:	https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/using-messagegroupid-property.html
[5]:	https://twitter.com/timbray/status/1246157403663388672?s=21
[6]:	https://stackoverflow.com/a/6821925
[7]:	https://youtu.be/h46IquqjF3E
[8]:	mailto:sudhir.j@gmail.com
[9]:	https://twitter.com/sudhirj