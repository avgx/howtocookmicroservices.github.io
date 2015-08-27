---
layout: post
title:  "Rails app with docker-compose - video"
short_title:  "Rails with Docker and docker-compose"
date:   2015-08-27 23:09:49
meta_description: ""
hide_in_sidebar: true
duration: '4m'
sections:
    - 'Videos / Screencasts'
    - Development Environment
redirect_from:
    - "updates/11-ruby-on-rails-app-with-docker-and-docker-compose.html"
---

hey what’s up guys, after watching this video you'll have an idea of what it takes to run a Rails app with docker and docker-compose.

<iframe width="640" height="360" src="https://www.youtube.com/embed/LAnJ1O4tgx0?rel=0" frameborder="0" allowfullscreen></iframe>

(Sorry for aspect ratio, going to use proper resolution on retina display next time! (facepalm))

### Commands executed as part of this video

#### Clone rails app
{% highlight bash %}
git clone git@github.com:akurkin/lightweight_rails_app.git
{% endhighlight %}

#### Create docker-machine on local with virtualbox provider
{% highlight bash %}
docker-machine create -d virtualbox --virtualbox-memory 2048 lra
{% endhighlight %}

#### Configure docker client to connect to new docker host
{% highlight bash %}
docker-machine env lra
eval $(docker-machine env lra)
{% endhighlight %}

#### Check docker host info
{% highlight bash %}
docker info
{% endhighlight %}

#### Launch services using docker-compose
{% highlight bash %}
docker-compose -f docker-compose.development.yml up -d
{% endhighlight %}

#### Check running containers
{% highlight bash %}
docker-compose -f docker-compose.development.yml ps
{% endhighlight %}

#### Create database, load schema and seed it
{% highlight bash %}
docker-compose -f docker-compose.development.yml run web rake db:create db:schema:load db:seed
{% endhighlight %}

#### Get ip address of docker host
{% highlight bash %}
docker-machine ip lra
{% endhighlight %}

#### Run specs in a container
{% highlight bash %}
docker-compose -f docker-compose.development.yml run -e RAILS_ENV=test web rake db:create db:migrate
docker-compose -f docker-compose.development.yml run -e RAILS_ENV=test web rspec
{% endhighlight %}

