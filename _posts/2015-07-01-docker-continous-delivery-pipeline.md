---
layout: post
title:  "Continuous Delivery with Docker"
short_title:  "CI/CD with Docker"
date:   2015-07-01 23:09:49
in_progress: true
sections:
    - 'Continuous Delivery'
redirect_from:
    - "updates/09-continuous-delivery-with-docker-first-notes-added.html"
---

hey what’s up guys, after reading this post you will know **how to build continuous delivery pipeline with docker**, building pieces of it and necessary steps required to **implement** it the **right way**!

## TL;DR
{:.no_toc}

If you want to know how to easily build something like this, subscribe and read on!

#### WHAT'S COMING:
{:.no_toc}

- Overview (*added on 03-07-2015*)
- Tools (*added on 03-07-2015*)
- GO.CD Intro
- Setup of GO.CD on local using docker-machine
- Setup of GO.CD Agents on local using custom boot2docker image
- Building custom boot2docker image (to run GO.CD agent and docker-compose)
- Configuration of pipelines (cover Dockerfiles, docker-compose etc)

![Continuous Delivery with Docker](/images/docker_continuous_delivery/gocd-main-screen-pipelines.png)

* Table of contents
{:toc}

## As a developer, I want

- … to **trigger specs** execution after pushing commits to a branch
- … to **automatically run Acceptance tests** using headless browser after successful specs run
- … to **provision environment** and **deploy platform** once story is ready for acceptance criteria verification!

This sort of automation is vital for successfull product development to save time on many manual tasks by automating them and reducing chance of errors.

## Tools

To build Continuous Integration/Continuous Delivery pipeline and support platform built of Microservices we would really need 3 parts to start with:

- **Continuous Integration software** - to automate our routine tasks: run jobs and then trigger another jobs and then another jobs and so on
- **Docker Distribution** - software to manage our images
- **Container Orchestration tool** -  to manage our containers. For simplicity we stick to Docker’s Orchestration Tools, but there are lot of options available to sort all possible needs.

### Continuous Delivery Software

One of the main requirements we are looking for is the need to easily work with concept of pipeline. Our tool of choice is go.cd from Thoughtworks, primarily for it’s absolutely incredible ability to construct complex pipelines out of small and simple pipelines (with Fan-In and Fan-Out dependencies) which will make it possible to trigger one job after successful execution of another job. More on this later.

### Docker Images Distribution

Main purpose of the docker distribution is to store and ship images. We will go with docker’s open-source distribution tool, but there’re alternatives available, such as:

- Docker’s hosted registry with private repositories
- Private Docker distribution deployed to your cloud
- Google Container Registry https://cloud.google.com/tools/container-registry/
- and so many more that it’s hard to cover all of them in one list

### Containers Orchestration Tool

For simplicity we stick to Docker Machine for creation of docker host. After that, Docker Compose will help us manage containers by allowings us to bootstrap environment with few containers in less than a minute.

