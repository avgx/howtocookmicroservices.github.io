---
layout: post
title:  "Platform Conventions"
date:   2014-11-01 23:40:00
sections:
    - 'Microservices Notes'
    - 'Implementation Details'
redirect_from:
    - "updates/08-platform-conventions-first-notes-added.html"
---

Conventions are very important, especially on early stages of development, because they establish patterns that you’ll have throughout the system. Not doing it now may cause a lot of re-work in the future. 

We’ll highlight few areas where conventions are important:

- routing
- microservice configuration
- naming

## Routing

As we've outlined in our architecture diagram for platform built of microservices we have 5 small apps that talk to each other and also open for clients, we'll need to define simple and clear URL structure that will be used platform-wide. This may sound too simple and obvious, but that's a small detail that need to be talked thru and agreed on.

#### CORS

Taking into account **CORS**, all requests for web clients should be coming from **one domain** to simplify and optimize performance (by avoiding HTTP OPTIONS request).

Here's a **couple of requirements**:

- we should be able to **easily determine** to what **service** request belongs to
- for **configuration of loadbalancer** we would need to have set of consistent rules between services

To solve requirements above, here's proposed URL structure:

http://example.com/**[service_identifier]**/**[service specific path]**

#### Example

- http://example.com**/catalog_service/**products/123
- http://example.com**/user_service/**users/3341
- http://example.com**/orders_service/**orders/99812

As you can see, this is super simple URL structure that gives you one main advantage when building web app - now you don't have to worry about CORS problem and actively manage configuration for every service. Everything goes thru load balancer, and load balancer can determine whether request belongs to catalog, user or orders services.

At the same time it makes it possible to have same path inside service, i.e. "common" endpoints for deployed version and service status for example, would be available at:

- http://example.com**/catalog_service/version**
- http://example.com**/user_service/version**
- http://example.com**/orders_service/version**

## Configuration

Configuration of services is also an important thing. You want to be able to:

- **preserve same format** of configuration **between** different **services**
- maintain same way of loading configuration

To solve that problem, very handy to have a shared component that will look for configuration files in one particular place and will pick them up for processing. More on componentization in respective chapter.

If you go with Rails, you can use a **config_for** feature available in [Rails 4.2+](http://api.rubyonrails.org/classes/Rails/Application.html#method-i-config_for)!

## Naming

You may think this is self-evident "convention" to raise here, but still, I'd like to mention. Everything related to naming sometimes hard. Below is a simple agreement on naming that helped us build clear and concise structure for platform built of microservices:

- all repositories related to platform should **start with platform's name**, for example "mia"
- **append service** to the microservice app, for example "mia_catalog_service"
- for **clients** - keep it simple and name it same as service, but **take out "service" suffix**, for example "mia_catalog". Main reason for that it's shorter and client of the service is never used inside service it tries to connect to
- client's module name should be unique. It'd be simpler if you'd call it "MIA::Catalog" for catalog client.

