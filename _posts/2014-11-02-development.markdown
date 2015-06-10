---
layout: post
title:  "Development Environment with Vagrant"
short_title: "Development with Vagrant"
date:   2014-11-01 23:10:00
meta_description: "Microservices development. Talks about setting up VM and containers for development of distribute system built using microservices architecture."
sections:
    - 'Development Environment'
redirect_from:
    - "updates/04-development.html"
    - "development.html"
---
Easy thing to draw diagrams and explain how it all supposed to work. It becomes slightly harder when it comes to real development work and establishing development approach in a big team.

## LYRICAL RETREAT
{:.no_toc}

Setting up 5 services locally on your laptop, configuring nginx, mysql, memcache and rabbitmq - not the most convenient way to do things here. Also, 5 is just in our prototyp-ish kinda platform, but imagine real system with few times more applications running.

As of this writing I've been using Vagrant for all of my development work for few years now and also for this MIA platform, and I feel myself confident with this tool. It is by no means one of the most important things in my development workflow. This post is intended to be "share experience" kinda thing, and given I have experience working with **Vagrant + Chef** I'll explain that one first, and then we'll jump onto **Docker + Fig**.

## DEVENV ARCHITECTURE
{:.no_toc}

Our requirements are:

- I want to make it as easy as possible to "get up and running"
- I want platform running in one environment
- In few minutes time I want to spin up new development environment

Changes to the actual architecture:

- microservices connected to **same MySQL** instance, only database name will be different
- microservices connected to **same memcache** instance, again, namespace will be different for each microservice
- (and obviously all of the microservices will be located in one VM)
- nginx acts as a loadbalancer dispatching URLs to appropriate services

![devenv architecture](/images/development/development.png)

## VAGRANT + CHEF
{:.no_toc}

After reading this post, you'll be able to quickly bootstrap development environment using VM:

- built with vagrant 
- provisioned with Chef
- custom chef recipe to install and setup our microservices.

## CONTENTS
{:.no_toc}

* Table of contents
{:toc}

Our solution consists of encapsulating everything into VM using [Vagrant](https://www.vagrantup.com) and automating provisioning using [Chef Solo](https://docs.vagrantup.com/v2/provisioning/chef_solo.html). 

There're 3 general "milestones" we'll need to deal with:

- VM Setup, i.e.
    + shared folders
    + shared ports
    + other basic configuration
- VM Provisioning to propagate infrastructure changes, i.e. following packages:
    + MySQL
    + Memcache
    + RabbitMQ
    + Nginx
- VM Provisioning to provision platform:
    + load balancer rules for nginx
    + cloning of microservices into VM
    + microservices configuration (i.e. config/unicorn.rb)
    + microservices start up scripts

First 2 are quite straightforward and easy, we'll touch them briefly, 3rd step is much more interesting and has room for creativity.

### 1. VM SETUP

This post is not about learning Vagrant and explaining what that is, so to a certain degree you should be familiar with Vagrant. Below some links that will help you cover that topic if you're a newcomer to Vagrant (or maybe just heard about it, but never played with). But first - requirements.

#### requirements

- synced folder
    + `../web` to `/web` - mapping "web" directory to the root "/web" inside VM
- port forwarding
    + guest 80 -> 8181 (LB and frontend, external API)
    + guest 3010 -> 3010 (*user service*)
    + guest 3020 -> 3020 (*catalog service*)
    + guest 3030 -> 3030 (*cart/checkout service*)
    + guest 3040 -> 3040 (*orders service*)
    + guest 3050 -> 3050 (*inventory service*)

#### links

- [Download Vagrant](https://www.vagrantup.com/downloads.html)
- [Install Vagrant](https://docs.vagrantup.com/v2/getting-started/index.html)

Ok, after you've got Vagrant installed make sure to setup VM according to requirements above.

#### setup steps
Now, let's init new Vagrant box:

~~~
mkdir -p ~/Development/mia/devenv
cd ~/Development/mia/devenv
vagrant init
~~~

Vagrantfile has been placed into `devenv` directory. Now we need to perform some changes to following configuration values, I'm just going to briefly list them up here, and then you can go and review results in github

- set `config.vm.box` and `config.vm.box_url`
- set `config.ssh.forward_agent` to `true`
- set `config.vm.synced_folder` for 'web' directory. Make sure to create this directory. It must exist before provisioning, otherwise it will fail.
- potentially you may want to increase RAM size for this VM, to do that use `vb.customize ["modifyvm", :id, "--memory", "2048"]`

Here's how our tree looking like before provisioning:

~~~
.
├── devenv
│   └── Vagrantfile
└── web
~~~

Ok, we are good to finish first step of our setup and build this box by triggering:

~~~
vagrant up
~~~

#### verification
Good, it's up and running now, so I can ssh to verify and then test my connection to Github.

~~~
vagrant ssh
ssh -T git@github.com
~~~

And you should see something like `Hi <your username>! You've successfully authenticated, but GitHub does not provide shell access.`

#### troubleshooting
In case you weren't able to correctly authenticate on Github, can be few reasons:

- Vagrantfile doesn't have `config.ssh.forward_agent` set to `true`
- SSH identity missing from your host machine (be careful, it's removed HOST machine restart). To check this, run:

~~~
ssh-add -l
~~~

   If you don't see any entities there, add one:

~~~
ssh-add -k <path to private key>
~~~

### 2. VM PROVISIONING INFRASTRUCTURE

Good, you've got bare VM running and now let's customize it for generic Ruby/Rails development. You'd need quite a few packages and some vagrant plugins. 

#### Eseential Vagrant plugins

- [vagrant-omnibus](https://github.com/opscode/vagrant-omnibus) - ensures the desired version of Chef is installed via the platform-specific Omnibus packages
- [vagrant-berkshelf](https://github.com/berkshelf/vagrant-berkshelf) - provides seamless integration with Berkshelf

~~~
vagrant plugin install vagrant-omnibus
vagrant plugin install vagrant-berkshelf
~~~

#### Packages

- rbenv
- memcached
- mysql
- rabbitmq
- nodejs
- [httpie](https://github.com/jakubroztocil/httpie)  (just because it's an awesome command-line app for making http requests!)

#### Berkshelf

To solve this problem, I recommend using [Berkshelf](http://berkshelf.com) which is absolutely amazing tool that takes all complexity of cookbooks management and deals with it in a very nice way. Also it has vagrant plugin which I briefly described above. Ok, let's get started. Create new file named Berksfile and put it in root directory of devenv. vagrant-berkshelf plugin will use that filename by default, if that file exists and install all necessary cookbooks.

~~~
source "https://supermarket.getchef.com"

cookbook "memcached", "1.7.2"
cookbook "mysql", "5.6.1"
cookbook "nginx", "2.7.4"
cookbook "nodejs", "2.2.0"
cookbook "rabbitmq", '3.4.0'
cookbook "rbenv", git: "git://github.com/fnichol/chef-rbenv.git", ref: "v0.7.2"
cookbook "httpie", "0.1.0"
~~~

If you were to bring your machine up, you'd see information about cookbooks being vendored. But before we actually bring it up, we'll specify some more information for chef to run.

Next thing would be to define VM provisioning with chef-solo provisioner, here's an excerpt from Vagrantfile with specific steps:

~~~
config.vm.provision "chef_solo" do |chef|
  chef.add_recipe "apt"
  chef.add_recipe "build-essential"

  chef.add_recipe "ruby_build"
  chef.add_recipe "rbenv::user"
  chef.add_recipe "rbenv::vagrant"

  chef.add_recipe "memcached"
  chef.add_recipe "nginx"

  chef.add_recipe "nodejs"
  chef.add_recipe "nodejs::npm"

  chef.add_recipe "mysql::server"
  chef.add_recipe "mysql::client"

  chef.add_recipe "rabbitmq::default"
  chef.add_recipe "rabbitmq::mgmt_console"

  chef.add_recipe "httpie"

  chef.json = {
    'mysql' => {
      'server_root_password' => 'root',
      'server_debian_password' => 'vagrant',
      'server_repl_password' => 'root',
      'allow_remote_root' => true,

      'client' => {
        'packages' => ['mysql-client', 'libmysqlclient-dev', 'ruby-mysql']
      }
    },

    'rbenv' => {
      'user_installs' => [
        {
          'user' => 'vagrant',
          'rubies' => ['2.1.0'],
          'global' => '2.1.0'
        }
      ]
    },

    'nodejs' => {
      'npm' => '2.1.15'
    }
  }
end
~~~

Now, after running `vagrant up --provision` you should be able to get VM running, and it should be fully suited for running any rails app backed by MySQL.

### 3. VM PROVISIONING MIA

We want to extend provisioning with a custom Chef recipe that will do following for us:

- install loadbalancer config
- install each microservice's start up script
- run bundle install
- bootstrap env with initial data

There's quite a few good articles on this topic, I'll provide links below under "Resources" part.

But for now, I'll leave it open until we really have couple of services to install. If you already do have something to install, below is a list of articles that helped me prepare custom MIA cookbook to provision platform.

## RESOURCES

- [Vagrant](https://www.vagrantup.com) - Create and configure lightweight, reproducible, and portable development environments.
- [Berkshelf](http://berkshelf.com) - Manage a Cookbook or an Application's Cookbook dependencies.
- [vagrant-omnibus](https://github.com/opscode/vagrant-omnibus) - A Vagrant plugin that ensures the desired version of Chef is installed via the platform-specific Omnibus packages.
- [vagrant-berkshelf](https://github.com/berkshelf/vagrant-berkshelf) - A Vagrant plugin to add Berkshelf integration to the Chef provisioners

List of articles dedicated to getting Vagrant Up and running as development environment

- [Using Vagrant and Chef For Reproducible, Isolated Rails Development Environments](http://gofullstack.com/articles/using-vagrant-and-chef-for-reproducible-rails-development-environments.html)
- [A better workflow with Chef & Vagrant](http://peterjmit.com/blog/a-better-workflow-with-chef-and-vagrant.html#berkshelf)
- [VAGRANT WITH CHEF PROVISION](http://codeispoetry.me/index.php/vagrant-with-chef-provision/)

