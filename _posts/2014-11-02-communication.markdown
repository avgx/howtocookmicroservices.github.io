---
layout: post
title:  "Inter-service Communication"
date:   2014-11-01 23:34:59
meta_description: "Microservices inter-service communication. Covers sync and async styles of communication using HTTP + REST and lightweight messaging respectively"
sections:
    - 'Microservices Notes'
    - 'Implementation Details'
redirect_from:
    - "updates/03-communication.html"
---

Communication between services is interesting, you have so many options to select from. We’ll cover the most straightforward and cheap from development perspective for now, but also mention few alternatives to that.

Let’s split our inter-service communication in 2 types:

- synchronous
- asynchronous

## Synchronous - REST HTTP

Simplest possible solution for synchronous communication between services is to apply same approach for communication as to end clients - 
**JSON over HTTP**. And probably it's fine to pay the price of a small overhead for ability to re-use endpoints by different services and share them with clients.

And of course if you need something more performant for "service to service" requests (i.e. user's authentication token verification), you can select one of options below:

- [protobuffs' ruby gem](https://github.com/localshred/protobuf)
- [messagepack](http://msgpack.org/); [msgpack-ruby](https://github.com/msgpack/msgpack-ruby)

## Asynchronous - Lightweight messaging

To support lightweight messaging, you'd need to select software package that will act as lightweight message broker delivering your messages to consumers running on respective microservices. There's a great variety of tools to support that, amongst them are:

- [RabbitMQ](http://www.rabbitmq.com); [bunny gem](http://rubybunny.info)
- [Redis](http://redis.io/); [Sidekiq](https://github.com/mperham/sidekiq), [Resque](http://resquework.org) gems
- [0mq](http://zeromq.org); [rubygem 0mq](https://github.com/jemc/0mq)
- many others, that you can review here [queues.io](http://queues.io)

I have to say, we've used RabbitMQ along with Ruby Bunny gem and absolutely amazing [serverengine](https://github.com/fluent/serverengine) that helped us build flexible and robust PUB/SUB communication between microservices.

Now let's cover 2 types of asynchronous communication and which one to use on example. 

#### Example
Imagine we have inventory service that either monitors file on FTP from warehouse or receives a request from warehouse about incoming SKU. When we get new information about SKU, few things should happen:

- record new numbers for SKU
- recalculate number of available items for sale for SKU
- let catalog know about SKU availability
- update user's wishlist for SKU

#### Choreography

It's better to share small diagram that may be worth thousand words.

![service choreogrpahy](/images/communication/choreography.png)

As you can see **one type of message** produced to **all queues** - *sku_processed*. It doesn't explicitly say that sku is available or not, or do we need to notify wishlist owners or no. Interesting thing next. *sku_processed* message only contains SKU identificator, either code or primary key. 

Now, respective consumers responsible for reaching out to inventory service, checking SKU's availability and performing domain specific operations:

- inventory service responsible for performing some extra calculations which weren't necessary to perform synchronously
- catalog service responsible for marking SKU as available or not available based on SKU availability from inventory service
- user service responsible for sending email notification about "wishlisted" item becoming available

You can reach this effect by:

- making inventory service aware of all queues it needs to update on incoming SKU
- using message broker's capabilities for "automatic" dispatching across right set of queues

For example, you can use RabbitMQ's *fanout exchange*, which is effectively sends copy of message to all "requested" queues, eventhough in your code you only need to produce 1 message to RabbitMQ *fanout exchange*. You can read more about it here [AMQP Concepts](https://www.rabbitmq.com/tutorials/amqp-concepts.html).

#### Orchestration

Let's review opposite to orchestration approach. On the diagram below, you'll see that inventory service keeps knowledge about things that should happen when new SKU arrives and moreover, inventory service is responsible for triggering these messages. 

Orchestration style provides a very different angle on this problem as opposed to sending "new sku arrived" message to all consumers (as in Choreography style), we have to send 3 messages "recalculate for sku", "update sku availability" and "update wishlists for sku". 

![service orchestration](/images/communication/orchestration.png)


### Summary
I really like an idea of **Choreography** style, as it encourages loose coupling without violating business domain's borders. As producer of the message doesn't have to know what other service supposed to do, it just provides an event, to which consumers may respond or no.

## Resources

- [RabbitMQ](https://www.rabbitmq.com/)
- [queues.io](http://queues.io) - comparison list of available packages to support messaging in one place
- [serverengine ruby gem](https://github.com/fluent/serverengine); [slides](http://www.slideshare.net/treasure-data/rubykaigi-2014-serverengine) - framework for implementing robust multiprocess servers like Unicorn
- [sneakers ruby gem](http://jondot.github.io/sneakers/)
