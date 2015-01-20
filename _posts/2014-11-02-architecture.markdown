---
layout: post
title:  "Architecture Pattern"
date:   2014-11-01 23:50:00
redirect_from:
    - "updates/01-microservices-architecture-pattern.html"
---

Hello, this is an introductory post to a series of articles about using Ruby on Rails and other Ruby frameworks to build a platform (let's call it MIA) empowered by Microservices Architecture. One more thing to mention right from the beginning - I am creating this to share my experience of building platform with Ruby on Rails and microservices architecture which involved a lot of practical decisions (unsurprisingly).


### What is microservice? {#microservice}

"Microservice" is a buzz word for a long time now, and I’m not going to try to explain to you what that is, it’d be better if you just put this aside for a little while and read this article from one of the prominent minds of the industry, Martin Fowler, to get yourself familiar with ideas around microservices: 

[Martin Fowler's Microservices](http://martinfowler.com/articles/microservices.html)


### Why? {#why}

- few small agile teams (co-located in different offices + remote team) working on multiple products
- failure isolation (yes, you’ll be greatful when it comes to debugging issue in production, and you have clear boundaries of each service responsibilities)
- decentralized data management
- super fast deployments to independent components (microservices)
- scalability
- flexibility

Those items above are not theories broken out for reading yet one more time. This is exactly what we've experienced whilst getting microservices off the ground and launching in production.

### How? {#how}
Now as we are getting closer to "how" part, I’ll mention that we've built this platform using Ruby language and focusing primarily on Ruby on Rails framework, but of course there are few more options that may be a great fit in this particular architecture. We’ll cover them over time.

And now onto exciting "what" part of this idea. We are building an **e-commerce platform**{:.inverse-highlight} that:

- talks JSON over HTTP
- frontend agnostic (anything can consume data, because it’s just an API):
    - web frontend (javascript client)
    - iOS app
    - Andriod app
    - third-party clients

### Architecture {#architecture}

Alright, let's define scope of our platform. Imagine simple e-commerce website with traditional (and quite limited!) set of functions:

- user accounts
- catalog
- cart and checkout
- orders processing
- inventory

### Monolith's Heritage

Architecture in traditional rails app you can describe in one picture (slightly simplified of course):

![monolith's heritage](/images/architecture/monoliths_heritage.png)
Picture 1.1

ALL requests that user makes follow this pattern:

- request delivered to rails app
- rails app reaches out to memcache to see if there's cached version of that resource
- if no, then it makes query to get resource from database

Simple, easy, not so scalable. From both: development and performance perspectives.

Imagine 30 people working on same rails codebase - nightmare! I bet in effective agile environment, they'll have quite a few problems and these will be amongst them:

- slow and clumsy releases
- hard to manage incoming changes
- lack of continuous delivery
- struggling with time it takes to manage codebase and keep all in sync

### Microservices Way

Here goes simplified version of architecture for same application built using microservices:

![microservices](/images/architecture/microservices-1.png)
Picture 1.2

As you can see on the image, microservice is a completely independent application that:

- talks json
- logically (and ideally physically) isolated from others
- doesn't share resources with other apps
- has it's own codebase
- has clearly defined data ownership

This approach provides quite a few **advantages at scale**{:.inverse-highlight}:

- independent deployments
- issues are easier to diagnose and debug
- more maintainable codebase
- more testable application (but we'll cover testing in subsequent chapters)

### Resources

- [Martin Fowler's Microservices](http://martinfowler.com/articles/microservices.html)
