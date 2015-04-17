---
layout: post
title:  "Docker Compose & Docker Machine"
date:   2014-11-01 23:09:50
sections:
    - 'Development Environment'
    - 'Continuous Delivery'
redirect_from:
    - "updates/05-docker-compose.html"
    - "updates/06-docker-compose-updates.html"
---
## TL;DR
{:.no_toc}

- install `docker-machine`
- install `docker-compose`
- create new machine with virtualbox driver
- deploy containers to newly created machine using `docker-compose`
- profit

## Intro
hey guys, in this little article we’ll talk about using **docker**, **docker-compose** and **docker-machine** for bootstraping development environment of the platform built with **microservices architecture**.

And then, right after that we'll talk about how to move it even further to provisioning environment in few seconds to remote machines (for QA/Automated Acceptance testing and for whatever else you can imagine).

We'll go thru a little bit of theory first, then we explain what do we actually build and then we put our hands on into building this from 0 by creating machine on local environment and then deploying our containers there.

* Table of contents
{:toc}

### Objective
Ok, let’s start with few goals for this project

* Learn and understand how tools work
    1. Docker
    1. Docker Machine
    1. Docker Compose
    1. Consul & Consul-template
* Make platform run on Docker Compose
* Enable Service discovery using Consul and Consul-Template
* Configure platform
    1. to run for development
        * be smart and pragmatic to **re-use same gems** installed on multiple services (using volumes)
        * **code** must be **mounted to container**, so that changes will instantly appear in container
    1. to run as QA/Testing environments (by testing environment here we mean execution of automated acceptance tests)
        * make **containers portable**
        * run **bundle install** as part of build process
        * cover further enhancements we can make to improve our Continuous Integration and Continuous Delivery flow
        * outline what Continuous Delivery flow would look like

### Tools

Let's start with tools. We'll briefly touch on these tools with relevant links to documentation or good resources (which I keep coming back to when needed).

#### Docker

"An open platform for distributed applications for developers and sysadmins."

**Why?** Portability and faster delivery.

- [Official 'Why Docker?' - Reference](https://docs.docker.com)
- [Docker Cheat Sheet](https://github.com/wsargent/docker-cheat-sheet)
- [Dockerfile - Reference](https://docs.docker.com/reference/builder/#usage)
- [Eight Docker Development Patterns](http://www.hokstad.com/docker/patterns)

#### Docker Machine
Docker Machine is part of Docker's official set of orchestration tools. It allows you to "manage docker hosts on your computer, on cloud providers and inside your own data center".

We will use **Docker Machine** to create docker host on our local **Mac OS X environment**. As next step we'll be able to use it to create remote machines using many popular cloud providers (full list available on documentation page)

- [Docker Machine - Reference](https://docs.docker.com/machine/)

#### Docker Compose

It's a command-line tool that allows you to automate configuration and management of containers for complex applications running with Docker.
It works by processing configuration file that you define (i.e. docker-compose.yml), and makes it possible to bootsrap full suite of your services in one command: `docker-compose up`

But you get to see how it works in a bit!

- [Docker Compose - Reference](http://docs.docker.com/compose/)
- [Docker Compose File - Reference](http://docs.docker.com/compose/yml/)

### Architecture Diagram
> An image worth thousand words (c)

![devenv architecture using docker-compose](/images/development/docker-compose.png)

Here’s our diagram of the platform running on docker-compose, now let me explain it.

Every box is a **container**.

There are **2 main logical groups of containers**: 

  - our **microservices** (on the diagram inside light grey colored box) 
  - other **'infrastructure' services** like database, elastic, memcache and rabbitmq (on the diagram inside dark blue box). These are built and deployed straight from image pulled from official Docker registry. So nothing fancy.

(I figured combining containers into logical groups would help picture this integration easier, as having arrows from every microservice container to 'infrastructure' service container will make diagram cluttered so it'll be hard to understand)

**Containers running microservices** are connected to **“infrastructure”** service **containers** using Docker Compose’s **links** instruction, which automatically links one container to another and enables sharing connection strings via environment variables, so in our service `config/database.yml` file you’ll specify connection details using environment varaibles, something like this:

{% highlight yaml %}
#
# ...
#
development:
  <<: *default
  database: mia_user_service_<%= ENV['RAILS_ENV'] %>
  host: <%= ENV['WEB_DB_1_PORT_3306_TCP_ADDR'] %>
  port: <%= ENV['WEB_DB_1_PORT_3306_TCP_PORT'] %>
#
# ...
#
{% endhighlight %}

The same connection principle applies to how all microservice containers connect to infrastructure service containers, simple.

Another way how our microservice containers will **discover each other** is via **Consul** and **Consul-Template**. We'll see how each container lets Consul know about it's availability to serve requests via **registrator**. Also load-balancer container that is running nginx will be updating live when microservice container becomes available or shuts down. 

Exciting thing to talk about!

# Docker Machine

OK before we start with all the setup, let's briefly talk thru Docker Machine.
Docker Machine will allow us to **quickly create host** and **install Docker** there. And also **configure** your local **docker client** to point to your machines. 

For now we'll be experimenting with localy created Docker Machine that pulls down and installs **boot2docker** for Mac OS X and then spins up machine with a given parameters (memory, disk size)

### Create local machine with VirtualBox driver

Notice that we pass following arguments:

- driver
- virtualbox-memory
- virtualbox-disk-size
- **name** of the new machine

{% highlight bash %}
$ docker-machine create -d virtualbox --virtualbox-memory "4096" --virtualbox-disk-size "32000" mia
{% endhighlight %}

### Configure Docker client
{% highlight bash %}
$ eval $(docker-machine env mia)
{% endhighlight %}

### Verify

To list all available machines and see current **active** machine run `ls` command

{% highlight bash %}
$ docker-machine ls
NAME   ACTIVE   DRIVER       STATE     URL                         SWARM
mia    *        virtualbox   Running   tcp://192.168.99.100:2376

$ docker-machine inspect mia
{
  "DriverName": "virtualbox",
  "Driver": {
      "MachineName": "mia",
      "SSHPort": 49490,
      "Memory": 4096,
      "DiskSize": 32000,
      "Boot2DockerURL": "",
      "CaCertPath": "/Users/devci/.docker/machine/certs/ca.pem",
      "PrivateKeyPath": "/Users/devci/.docker/machine/certs/ca-key.pem",
      "SwarmMaster": false,
      "SwarmHost": "tcp://0.0.0.0:3376",
      "SwarmDiscovery": ""
  },
  "CaCertPath": "/Users/devci/.docker/machine/certs/ca.pem",
  "ServerCertPath": "",
  "ServerKeyPath": "",
  "PrivateKeyPath": "/Users/devci/.docker/machine/certs/ca-key.pem",
  "ClientCertPath": "",
  "SwarmMaster": false,
  "SwarmHost": "tcp://0.0.0.0:3376",
  "SwarmDiscovery": ""
}
{% endhighlight %}

# Docker Compose

Ok, now that we have our machine up and running, we can deploy our containers there. For this, we would need to define how exactly our system would work. 

Here comes **Docker Compose**.

[Docker Compose - Documentation](http://docs.docker.com/compose/)

Below, is docker-compose.yml for our platform, it consists of the few docker images that are drawn on the diagram above, let me list them below along with their job:

1. **Load-balancer**
  + nginx (with a pretty-straightforward config based on route matching)
  + consul-template: updates nginx application config every time service becomes available/not available
1. **Consul**
  + tool for discovering and configuring our services
1. **Registrator**
  + "service registry bridge" that listens for container's availability and registers/deregisters them in Consul (we use Consul, but registrator supports many pluggable adapters for other service discovery tools like etcd, etc)

    Really good article of how to use **registrator** along with **Consul and Consul-Template** is available here:

    [Shane Sveller - Load-balancing Docker containers with Nginx and Consul-Template](https://tech.bellycard.com/blog/load-balancing-docker-containers-with-nginx-and-consul-template/)
1. Data-only containers
  + for database data
  + for RubyGems

    Because we want to have persisted ruby gems outside of our microservice too.
    
    Implementation is fairly straightforward:

      - have a data-only container with mountable volume
      - configure microservice's image that wants to use shared rubygems container using environment variables `GEM_HOME` and `BUNDLE_PATH` (make sure to update `PATH` with rubygems' /bin location)
      - mount data-only container to microservices container, using `volumes_from` instruction from docker-compose

    This proved to be viable idea for **development** environments, of course it hurts portability and doesn't make sense when deploying container in the wild for QA/Testing environments, we'll cover this in the future).

    Here's a good article that speaks on the topic:

      * [Phil Misiowiec - How to Create a Persistent Ruby Gems Container with Docker](http://www.atlashealth.com/blog/2014/09/persistent-ruby-gems-docker-container/) - explains of why this is an approach to go with, also have some guidelines and examples for docker without docker-compose, and also finishes with example for docker-compose

1. Database (MySQL)
1. Memcache
1. RabbitMQ
1. Elastic
1. **Microservice Containers**
  - Rails app running unicorn that is available on exposed port
  - Consul-Template service

        have consul-template to update specified file based on supplied template, once service becomes available/unavailable 
    
        **Alternative option would be:**
        **integrate** with consul **directly** and **query** consul for other services details (in that case you don't even need consul-template). But we are having consul-template here just because for backward compatibility with chef-based provisioning system (again this is taken from real life project)

  The most interesting 2 docker images of this setup are (we'll cover them below):

  - nginx load-balancer docker image
  - rails microservice docker image [available on github - akurkin/mia](https://github.com/akurkin/mia) 
  
  Why? Because rest of **services** we are having **are built from publicly avaialble docker images**, meaning we don't have to do any setup to make it work.

## Rails Microservice Dockerfile

[Rails Microservice Dockerfile on GitHub - akurkin/mia](https://github.com/akurkin/mia)

Above is a repository with image and respective services, but here I will just list specific instructions and comment on them (for brevity reasons).

Steps of the image build process:

- install some apt-packages (mysql client, runit, nodejs)
- install consul-template
- create service directories
- add shell scripts to service directories (for unicorn, consul-template and consumers). They are one-liners that execute given scripts from service repository
- set gem/bundle specific environment variables to install gems onto shared volume
- install bunder
- `ONBUILD` instructions to add Gemfile/Gemfile.lock and then add code

#### NOTES

- `COPY id_rsa_rosi /root/.ssh/id_rsa` - unfortunately this is a workaround to overcome difficulties with installing private ruby gems, since containers don't have any `secrets` or a way to use private git repos w/o adding key. This is something that hopefully will be fixed in the future
- set gem/bundle specific environment variables to point to shared rubygems volume
{% highlight ruby %}
ENV GEM_HOME /web/rubygems/2.0.0-p643
ENV BUNDLE_PATH /web/rubygems/2.0.0-p643
ENV PATH /web/rubygems/2.0.0-p643/bin:$PATH
{% endhighlight %}
- notice that `bundle install` is not part of the base Dockerfile for microservice, because as we said above for development environment our gems will be installed in a shared rubygems volume, and you can not mount volumes during your build process, as "it'll violate portability". 
- our web server (unicorn) will be running on `PORT` 3000 in all containers

## YAML configuration file

  Here's an example of `docker-compose.yml` file. Notice that we also have `common.yml` file which contains shared definition of the service.

  Another gotcha is that microservice container has exposed `PORT` 3000, but doesn't specify port on the host machine, this is done intentionally so that we can have many containers with the same exposed port running.

  Environment Variables used:

  - **SERVICE_3000_NAME**: name of the service in consul, can be used in consul-template queries
  - **SERVICE_3000_TAGS**: list of tags that will be also used in consul-template queries

[Full docker-compose.yml file - Gist](https://gist.github.com/akurkin/1d43fb03c6f415093bab)

Below only 3 of the platform microservices included in `docker-compose.yml` to simplify the example. See how microservice containers connect to infrastructure containers (via docker-compose `links`).

#### common.yml
{% highlight yaml %}
#
# Shared definition of ruby microservice
#
microservice:
  command: "runsvdir /etc/service"
  environment:
    PORT: 3000
    RAILS_ENV: development
    SERVICE_PLATFORM: "mia"
  ports:
    - 3000
#
# Shared definition of frontend js client
#
frontend:
  environment:
    SERVICE_PLATFORM: "mia"
  ports:
    - 80
{% endhighlight %}

#### docker-compose.yml
{% highlight yaml %}
#
# Load Balancer
#
lb:
  build: ./lb/
  links:
  - consul
  ports:
  - "80:80"
  - "8181:8181"
#
# Service Discovery - Consul
#
consul:
  command: -server -bootstrap -advertise 10.0.2.15
  image: progrium/consul:latest
  ports:
  - "8300:8300"
  - "8400:8400"
  - "8500:8500"
  - "8600:53/udp"
#
# Service Discovery - Registrator
#
registrator:
  command: -ip=10.0.2.15 consul://consul:8500
  image: gliderlabs/registrator:latest
  links:
  - consul
  volumes:
  - "/var/run/docker.sock:/tmp/docker.sock"
#
# Data-only container for rubygems
#
rubygems:
  image: "busybox"
  volumes:
    - /web/rubygems/2.0.0-p643
data:
  image: "busybox"
  volumes:
    - /var/lib/mysql
#
# Infrastructure
#
db:
  image: "mysql:latest"
  environment:
    MYSQL_ROOT_PASSWORD: root
  volumes_from:
    - data
memcache:
  image: "sylvainlasnier/memcached:latest"
rmq:
  image: "rabbitmq:management"
  hostname: "rmq"
  ports:
    - '15672:15672'
elastic:
  image: "elasticsearch:latest"
#
# Microservices
#
user:
  extends:
    file: common.yml
    service: microservice
  build: ./services/rosi_user_service
  environment:
    SERVICE_3000_NAME: "user"
    SERVICE_3000_TAGS: "backend,user"
  volumes:
    - ./services/rosi_user_service:/web/service
  volumes_from:
    - rubygems
  links:
    - db
    - memcache
    - rmq
    - consul
    - elastic
catalog:
  extends:
    file: common.yml
    service: microservice
  build: ./services/rosi_catalog_service
  environment:
    SERVICE_3000_NAME: "catalog"
    SERVICE_3000_TAGS: "backend,catalog"
  volumes:
    - ./services/rosi_catalog_service:/web/service
  volumes_from:
    - rubygems
  links:
    - db
    - memcache
    - rmq
    - consul
    - elastic
#
# Frontend
#
keeplounge:
  extends:
    file: common.yml
    service: frontend
  build: ./frontend/keep_lounge
  environment:
    SERVICE_80_NAME: "keeplounge"
    SERVICE_80_TAGS: "frontend,keep,keeplounge"
  volumes:
    - ./frontend/keep_lounge:/web/client
keepwww:
  extends:
    file: common.yml
    service: frontend
  build: ./frontend/keep_www
  environment:
    SERVICE_80_NAME: "keepwww"
    SERVICE_80_TAGS: "frontend,keep,keepwww"
  volumes:
    - ./frontend/keep_www:/web/client

{% endhighlight %}

## WHAT'S NEXT

- add Load Balancer Dockerfile
- add information about LB config
- make version of docker-compose.yml to be used on CI/QA environments
- write down how to implement Docker private registry on Mac OS X
- implement script that can be used to build and then push images to private registry
- Setup pipeline on Go Cd to watch for changes, then build and push images to private registry

## Reference & Resources
{:.no_toc}

- [Valueable Docker Links](http://www.nkode.io/2014/08/24/valuable-docker-links.html)
- [Phil Misiowiec - How to Create a Persistent Ruby Gems Container with Docker](http://www.atlashealth.com/blog/2014/09/persistent-ruby-gems-docker-container/)
- [Eight Docker Development Patterns](Eight Docker Development Patterns)
[Persistent Volumes - Data-only container pattern](http://container42.com/2013/12/16/persistent-volumes-with-docker-container-as-volume-pattern/)
- [DOCUMENTATION - Dockerfile](https://docs.docker.com/reference/builder/#usage)
- [DOCUMENTATION - Docker Compose](http://docs.docker.com/compose/)
- [Digital Ocean - Understanding Nginx http proxying and Load Balancing](https://www.digitalocean.com/community/tutorials/understanding-nginx-http-proxying-load-balancing-buffering-and-caching)
- [Load balancing docker containers with Nginx and Consul-Template](https://tech.bellycard.com/blog/load-balancing-docker-containers-with-nginx-and-consul-template/)

