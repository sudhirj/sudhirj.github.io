---
layout: page
title: What are Message Queues?
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
        content: Why are they useful?
        url: /little-guide-to-message-queues/why
---
 
Message Queues are a way to transfer information between two systems. This information – a message – can be data, metadata, signals, or a combination of all three. The systems that are sending and receiving messages could be processes on the same computer, modules of the same application, services that might but running on different computers or technology stacks, or entirely different kinds of systems altogether – like transferring information form your software into an email or an SMS on the cellphone network. 

The idea of a messaging system has been around a very long time, from the message boxes[^1] used for moving information between people or office departments, to telegrams, to your local postal or courier service. The messaging system in the physical world that comes closest to what we have in computing is probably the pnuematic tubes[^2] that moved messages through buildings and cities using compressed air until a few decades ago (and are still used in some places today).

The kinds of messages we want transferred today might be a note that something technical happened, like CPU usage exceeding a limit; or a business event of interest, like a customer placing an order; or a signal, like a command that tells another service to do something. The contents of each message will be driven entirely by the architecture of your application and its purposes – so for the rest of this book, we don't need to be concerned about what's inside a message – we're more concerned with how the message gets from the system where it originates (the *producer*, *source* or *sender*) to the system where's it's supposed to go (the *consumer*, *destination* or *receiver*). 

We need message queues because no system exists or works in isolation – all systems need to communicate with other systems in structured ways that they both can understand, and in controlled bandwidth that they both can handle. Any non-trivial process needs a way to move information between each stage of the process; any workflow needs a way to move the intermediate product between the stages of that workflow. Message queues are a great way to handle this movement. There are plenty of ways of getting these messages around using API calls, file systems, or many other abuses of the natural order of things; but all of these are ad-hoc implementations of the message queue that we sometimes refuse to acknowledge we need. 

The simplest mental model for a message queue is a tube that you can roll a ball into. You write your message on a ball, roll it into the tube, and someone or something else receives it at the other end. There are a lot of interesting benefits with this model, which we'll see the next chapter. 

---

[^1]:	Literally from where we get the words ‘inbox’ and ‘outbox’, used to designate the boxes for messages that were coming *in* for processing and going *out* after. 

[^2]:	The Wikipedia page is [here][1], but if you Google images of pneumatic tubes you can easily have your mind blown by how massive and complex these things were. 

[1]:	https://en.wikipedia.org/wiki/Pneumatic_tube