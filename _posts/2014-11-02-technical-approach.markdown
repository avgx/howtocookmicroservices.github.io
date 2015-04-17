---
layout: post
title:  "Technical Approach"
date:   2014-11-01 23:35:00
meta_description: "Microservices ruby and ruby on rails. Covers possible ways to jump onto breaking up monolith into microservices"
sections:
    - 'Implementation Details'
redirect_from:
    - "updates/02-technical-approach-added.html"
---

Ok, now as we are done with homework reviewing theory and approach. Let's jump onto practical part and figure out how we are going to build this. There’s a number of frameworks available but we’ll consider few:

- Ruby on Rails
- Sinatra
- Lotus (fairly new player)

There are PROs and CONs against each framework in particular. I’ll describe below what I’ve done in the past and few reasons to comment on that decision.

The most straightforward and “cheap” from development perspective way

### Rails + rails-api

Advantages:

- same framework in every microservice - kinda make sense when you are building product from scratch (MVP or prototype)
- unified way to plug in functionality that should be available in every service thru Railtie and gems mechanism
    - for example if you want same log formatter in every service, you build a railtie to set logger as part of config initializer in your shared gem, and you don’t need to do manual work in every service to install that logger
- consistency
- fastest way from 0 to hero

### The “right tool for the job”

Of course it goes without saying that in some cases Rails is an overkill with all of the available libraries …or you just want something more performant.

And of course there are couple of options:

- different Ruby framework (Sinatra, Lotus, etc)
- different technology (Go, Scala, etc)

One of the advantages of this approach is a real flexibility. And it goes w/o saying that with microservices architecture in place it never been easier to employ different technology and replace one service with something completely different (if you can justify that replacement).

### Componentization (via gems)

Ok, we’ve decided to go simplest and the most straightforward approach here - build rails apps that talks json.
But then the one of the earliest problems you’ll encounter - how to share code between multiple services.

Private ruby gems. Yes, elegant and quick.
One of the examples I always refer to, is that every service need to have a “status” and “version” endpoint. 
Putting another route or even controller in every service is an inefficient headache. We don't like ineffeciency and headaches. You can easily build a gem with required mountable engines and just include it into your service’s Gemfile, and "mount" into initializer of your application only once.

We'll talk more about shared libraries and common code in Componentization chapter.

