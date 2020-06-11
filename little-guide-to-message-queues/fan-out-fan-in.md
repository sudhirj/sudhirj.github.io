---
layout: page
title: Fan-Out & Fan-In
author:
    title: The Little Guide to Message Queues
    title_url: /little-guide-to-message-queues/
    external_url: false
    description: A short guide to what, why and wtf! of message queues.
page_nav:
    prev:
        content: Index
        url: /little-guide-to-message-queues/
    # next:
    #     content: Next page
    #     url: '#'
---

When building a distributed system there's often a need to have the same message sent to multiple receivers – besides the usual receiver of the message, we also often want the same message sent to other places, like an archiver[^kinesisfirehose], an audit log (for compliance and security checks) or an analyser for our dashboards. If you're using an event driven architecture with many services, you might want to use a single *event bus* in your application, where all the messages posted into this event bus are automatically sent to all of your services. This is called a *fan-out* problem, where a message from one producer needs to reach many consumers. 

[^kinesisfirehose]: Check out [AWS Kinesis Firehose](https://aws.amazon.com/kinesis/data-firehose/) for stuff like this. You can send your messages in at any rate, and Kinesis will batch into timestamped files and save them to S3 for pennies per gigabyte.

The inverse problem, where a single receiver is tasked with reading the messages posted to multiple queues is also common – in the example we considered above, a receiver that was archiving all messages or creating an audit log would probably receive all the messages generated in an organisation, on every queue. It's also common in service architectures to have a concern like notification handled separately – so a message about a new confirmed order might need to go to both a shipping queue and an email notification queue. This is a *fan-in* problem, where the messages from many producers need to reach the same consumer. 

If all the producers are putting their messages directly into queues, this would be a really difficult problem to solve – we'd have to somehow intercept our queues, and reliably copy the messages into multiple queues. Building, configuring and maintaining this switchboard simply isn't worth the time or the effort – especially when we could just use *topics* instead.

One way to think about topics is that they're similar to the headings you'd see on a notice board at a school or an office. Producers post messages under a specific topic on a board, and everyone interested in that topic will see the message. The most common way messaging systems send the messages to interested receivers is an HTTP(S) request, sometimes also called a *webhook*. In a push-based system like a HTTP request, the message is pushed into the receiving whether it's ready or not. This re-introduces the coupling that we talked about earlier which we want to avoid – we don't want a situation where our receiver collapses under the crushing load of tens / hundreds / thousands / millions of webhooks over a short span of time. The answer here, again, is to just use a message queue to soak up the messages at whatever rate they're generated. The receivers can then process them at their own pace. 

Automatically copying message from one queue into multiple queues isn't strictly a message queue feature, but it is complementary – most full-featured messaging systems will offer a way to do this. Producers will still continue to put messages into a single place as usual, but internally the messages will be copied to multiple queues, each of which will be read by their respective receivers. 

In AWS, the service that provides topic based messaging is the Simple Notification Service (SNS). Here you create a topic and publish messages into it – the API to publish a message into an SNS topic is very similar to that of publishing a message into an SQS queue, and most producers don't have to care about the difference. SNS then has options available to publish that message into any number of *subscribed* SQS queues (at no extra charge). Each of these subscribed SQS queues would then be processed by their respective receivers. 

If you're working with a different system like Apache Kafka, you'll see similar concepts there as well - you'll have *topics* that you publish messages into, and any number of consumers can each read all the messages in a topic[^kafka].
 
[^kafka]: Assuming they're all in different *consumer groups*, which would mean you have one consumer group per service. If all your Kafka consumers are in the same consumer group, that makes the Kafka topic behave like a plain SQS queue, where each message is delivered to any one agent of the consumer. I personally find the SNS + SQS split easier to work with, but Kafka has these concepts fully integrated offers a few extra features like ordering, at the expense of scalability.

This combination of these scenarios is common enough that there's a simple well established pattern to handle it: 
* Publish every message to one appropriate SNS topic.
* Create on SQS queue for each receiver.
* Subscribe each receiver's SQS queue to the SNS topics that the receiver is interested in.

Note that in order for this work, the receiver will have to be aware that it might process messages from multiple topics – we don't want the receiver crashing when it sees a message from a topic it doesn't understand. It makes sense to have all receivers swtich-case on the message type as soon as they receive it, and handle each recognized message type separately, with logging or alerts for unexpected messages.

Since it's possible to subscribe an SQS queue to any number of topics, there's no extra plumbing required at a receiver to process messages from multiple topics. And of course, it's possible to have any number of message queues subscribed to a single topic. This kind of setup supports both fan-out as well as fan-in, and keeps your architecture open to expansion and changes in the future. 

---