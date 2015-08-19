---
layout: post
title:  "Continuous Integration and Delivery with Docker Compose"
short_title:  "Continuous Integration with Docker"
date:   2014-11-01 23:09:49
meta_description: "Continuous Integration and Continuous Delivery of microservices with Docker and Docker-Compose using GoCD integration software. Guide on how to setup continuous delivery pipeline using docker compose"
sections:
    - 'Continuous Delivery'
redirect_from:
    - "updates/09-continuous-delivery-with-docker-first-notes-added.html"
    - "updates/10-continuous-delivery-with-docker-pipeline-diagram-added-and-gocd-intro.html"
---

hey what’s up guys, after reading this post you will know **how to build continuous delivery pipeline with docker**, building pieces of it and necessary steps required to **implement** it the **right way**!

* Table of contents
{:toc}


#### THE PLAN

- <del><b>Overview</b></del> (*added on 03-07-2015*)
- <del><b>Tools</b></del> (*added on 03-07-2015*)
- <del><b>Pipeline diagram</b></del> (*added on 10-07-2015*)
- <del><b>Update diagram to picture high-level pipeline from buidling an image to deploying it to remote environment</b></del> (*added on 18-08-2015*)
- <del><b>GO.CD Intro</b></del> (*added on 10-07-2015*)
- Setup of GO.CD on local using docker-machine
- Setup of GO.CD Agents on local using custom boot2docker image
- Building custom boot2docker image (to run GO.CD agent and docker-compose)
- Configuration of pipelines (cover Dockerfiles, docker-compose etc)

If you want to know how to easily build something like this, subscribe and read on!

![Continuous Delivery with Docker](/images/docker_continuous_delivery/gocd-main-screen-pipelines.png)

## As a developer, I want

- … to **trigger specs** execution after pushing commits to a branch
- … to **automatically run Acceptance tests** using headless browser after successful specs run
- … to **provision environment** and **deploy platform** once story is ready for acceptance criteria verification!

This sort of automation is vital for successfull product development to save time on many manual tasks by automating them and reducing chance of errors.

## Tools

To build Continuous Integration/Continuous Delivery pipeline and support platform built of Microservices (in fact any platform built with docker) we would really need 3 parts to start with:

- **Continuous Delivery software** - to automate our routine tasks: run jobs and then trigger another jobs and then another jobs and so on
- **Docker Distribution** - software to manage our images
- **Container Orchestration tool** -  to manage our containers. For simplicity we stick to Docker’s Orchestration Tools, but there are lot of options available to sort all possible needs.

### Continuous Delivery Software

One of the main requirements we are looking for is the need to easily work with concept of pipeline. Our tool of choice is [GoCD from Thoughtworks](http://www.go.cd/).

Primarily for it’s absolutely incredible ability to **construct complex pipelines out of small and simple pipelines** (with Fan-In and Fan-Out dependencies) which will make it possible to trigger one job after successful execution of another job and clearly define and visualize dependencies.

### Docker Images Distribution

Main purpose of the docker distribution is to store and ship images. We will go with docker’s open-source distribution tool, but there’re alternatives available, such as:

- Docker’s hosted registry with private repositories
- Private Docker distribution deployed to your cloud
- Google Container Registry https://cloud.google.com/tools/container-registry/
- and so many more that it’s hard to cover all of them in one list

### Containers Orchestration Tool

For simplicity we stick to **Docker Machine** for creation of docker host. After that, **Docker Compose** will help us manage containers by allowings us to bootstrap environment with few containers in less than a minute.

If you want to know more about our setup, check what we're doing for [Development environment with Docker and Docker Compose](/docker-compose/).

## Continuous Delivery Pipeline Diagram

![Continuous Delivery pipeline diagram with docker](/images/cd/cd_pipeline_overview.png)

This is a diagram of the continuous delivery pipeline. In the first box, we're getting code from remote repository. Then we execute specs,

### Run Unit Tests Pipeline walkthru


- Every box in diagram represents and individual task executed only if previous task was successful
- Pipeline is automatically triggered on incoming material change (i.e. when developer commits to repo).

![Unit specs pipeline](/images/cd/build_deployable_image_pipeline-horizontal.png)

#### TASK #1 - Checkout microservice code

In repository we have file **docker-compose.yml** which defines all microservice's runtime dependencies. For instance: mysql, elastic, memcached, etc.

This allows us quickly bootstrap it and execute any job we want,practically - run unit tests (via rspec in our case).

#### TASK #2 - Start container and execute specs

In practice, before you can do this, you need to **install dependencies if they are not bundled into your git repo**.

In Ruby world it means running `bundle install` and installing required gems. Also to mention: if you're going to run `bundle install` before every specs execution you may go crazy because of the time it takes to run it in an average ruby app.

To solve this we're running `bundle install` as **part of docker-compose with configured shared volume for gems**. What it means in reality is that every time you run `bundle install` it won't install gems if they were installed previously, even for another service.

This is only needed as an intermediate step to decrease time it takes from committing to execution of specs. In next task we'll prepare portable docker image that has dependencies bundled and ready to be deployed.

#### TASK #3 - Build portable docker image

What does it mean in this context?

It's very simple, on TASK #2, we've mentioned that we're executing dependency installation at runtime in order to make service running and execute specs.

In the current task we're running `bundle install` as **part of docker build process** to install dependencies into an image itself. After that an image is ready to be deployed anywhere and is prepared to run.

#### TASK #4 - Push image to docker registry

Tag and push an image prepared in previous task to the docker registry.

## Introduction to GoCD

**NOTE:** This is my personal subjective opinion on GoCD (compared to Jenkins).

Why did we switch to GoCD after many years of using Jenkins?

- **Pipelines**. You construct pipelines, that are acting as building blocks of more complex pipelines, making it easy to operate them and parallelize when needed.
- **Visualization** of pipelines. It is very **easy to compare** between different builds **what changed on a pipeline level** (this was crucial for us as Jenkins was making it impossible to see changes that broke automated acceptance tests suite if there were many repositories changed, again talking about case when platform built of many microservices)
- **Easy parallel execution** of tasks that can be split up (for example automated acceptance test suite that runs headless browser and takes quite a bit of resources)
- Easier **management and configuration**:
    - templates are vital when you work with many services. You can setup the pipeline once, extract template out of it and then create other pipelines based on that templates
    - configuration of the server is also available in the straightforward xml and is accessible from the browser

Below are list of resources that can give you an overview of why you should start using GoCD.

- [Why GoCD?](http://www.go.cd/learn-more/why-go.html)
- [How do I do CD with Go? - Thoughtworks](http://www.thoughtworks.com/insights/blog/how-do-i-do-cd-go-part-1-domain-model-concepts-abstractions)
- [GoCD - The right tool for the job?](http://thoughtworks.github.io/p2/issue11/go-cd-the-right-tool-for-he-job/)
