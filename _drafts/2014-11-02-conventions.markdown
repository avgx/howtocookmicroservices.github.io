---
layout: post
title:  "Conventions"
date:   2014-11-01 23:40:00
---

Conventions are especially important on early stages of development of MIA because they establish patterns that you’ll have throughout the system. Not doing it now may cause a lot of re-work in the future. 

We’ll highlight few areas where conventions are important:

- routing
- microservice configuration
- naming (service + client)
- API versioning

## Routing

As we've outlined in our architecture diagram for MIA we have 5 services that talk between each other and also open for clients, we'll need to define simple and clear URL structure that will be used platform-wide. This may sound too simple and obvious, but that's a small detail that need to be talked thru and agreed on. Also taking into account CORS, all requests for web clients should be coming from one domain to simplify and optimize performance (by avoiding HTTP OPTIONS request).

Here's a couple of requirements:

- we should be able to easily determine to what service request belongs to
- for easier configuration of loadbalancer we would need to have set of consistent rules between services

To solve requirements above, here's proposed URL structure:
http://howtocookmia.com/<service_identifier>/<service specific path>

#### Example

- http://howtocookmia.com/**catalog_service**/products/123
- http://howtocookmia.com/**user_service**/users/3341
- http://howtocookmia.com/**orders_service**/orders/99812

So that as you can see, super simple URL scheme structure that gives you one main advantage when building web app - now you don't have to worry about CORS problem and actively manage configuration for every service. Everything goes thru load balancer, and load balancer can determine whether request belongs to catalog service, user or orders, etc.

At the same time it makes it possible to have same path inside service, i.e. "common" endpoints for deployed version and service status for example, would be available at:

- http://howtocookmia.com/**catalog_service**/version
- http://howtocookmia.com/**user_service**/version 
- http://howtocookmia.com/**orders_service**/version

## Configuration

Configuration of services is also an important thing. You want to be able to:

- preserve same format of configuration between multiple services
- maintain same way to load configuration

To solve that problem, very handy to have a shared component that will look for configuration files in one particular place and will pick them up for processing. More on componentization in respective chapter.

## Naming

You may think this is self-evident "convention" to raise here, but still, I'd like to mention. Everything related to naming sometimes hard. Below is a simple agreement on naming that helped us build clear and concise structure for platform empowered by microservices architecture:

- all repositories related to platform should start with platform's name, for example "mia"
- append "service" to the microservice app, for example "mia_catalog_service"
- for clients - keep it simple and name it same as service, but take out "service" suffix, for example "mia_catalog". Main reason for that it's shorter and client of the service is never used inside service it tries to connect to.
- client's module name should be unique, don't lump everything onto global scope of MIA. It'd be simpler if you'd call it "MiaCatalog" for catalog client.

## API Versioning

