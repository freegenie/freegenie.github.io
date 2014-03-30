---
layout: post
title: zeromq, em and pipeline pattern
published: true
tags: blog programming zeromq
category: blog
description: In which I expose how I made zmq work with pipeline pattern
sharing: true
footer: true
comments: true

---
  
[0MQ](http://www.zeromq.org/) (zeromq) is a library that provides the very low level nuts and bolts 
to work with sockets (unix,tcp and more) and messaging in general. It's extremely lightweight and 
fast, with binding for almost any language. It's possible to pick from a set 
of predefined socket behaviours that cover the most common messaging patterns: 

* Request-reply pattern
* Publish/Subscribe pattern
* Pipeline pattern
* Exclusive pattern

After reading documentation and API docs, and after a couple of hours spent playing with it, 
well, it will be very hard for you to stop thinking on how you would rewrite
from scratch almost everything you did before. 

<!--more-->

In this article I'll focus on the _Pipeline pattern_ which is the main reason I stumbled on it, 
but it's just a matter of time and I'll dig into the other ones too. 

## The Pipeline pattern 

"How would you let a web app talk to a pool of eventmachine workers?". This is the problem to 
solve this time. One obvious solution would be to use a message broker. There's plenty of them 
around. I heard very good comments on [RabbitMQ](http://www.rabbitmq.com/), even though I never 
worked with it. Another good player seems to be [Redis](http://redis.io/) these days, with its
PUB/SUB feature. More recently I put my eyes on Mongodb's [capped collections](http://www.mongodb.org/display/DOCS/Capped+Collections)
which allows you to have _tailable cursors_, another possible solution in this scenario.

What I was looking for though was a way to just send a message from one process to another,
without having to setup another daemon. Having heard of this 0MQ thing to be a 
broker-less messaging system I decided to give it a try. 

Workflow is simple:

- webapp receives a request
- webapp pass the message to one eventmachine worker in the pool

Using a Pub/Sub pattern in this case would not be the perfect fit, since
all workers would receive a message, while in this case we want just one 
of them to be triggered and start the job.

I found the [Pipeline pattern](http://api.zeromq.org/2-1:zmq-socket) description in the API doc:

    The pipeline pattern is used for distributing data to nodes 
    arranged in a pipeline. Data always flows down the pipeline,
    and each stage of the pipeline is connected to at least one node.
    When a pipeline stage is connected to multiple nodes data 
    is round-robined among all connected nodes.

It seems to perfectly fit the case.

## Let's start coding

I love to prototype, so here is my proof of concept: 

<script src="https://gist.github.com/2013593.js"> </script>

Browsing around I found this github project [em-zeromq](https://github.com/deepfryed/em-zeromq), 
very well done tiny library with some convenience method to work the right way in EM.

## The Big Gotcha

Please note the __sleep 1__ row in _web.rb_. This got me stuck for some time and I wish I've read
the documentation better. That sleep line is required in order to ensure that all desired connections
are started before the application starts to send the first messge. If you don't do that, all pushes will be
delivered to the first defined connection, no metter if the worker on that socket is up or down.

## Running the examples

In order to run these examples, open four terminals and run:

{% highlight bash %}
    ruby web.rb
{% endhighlight %}

And then in other three terminals respectively: 

{% highlight bash %}
    port=4444 ruby server.rb
    port=4445 ruby server.rb
    port=4446 ruby server.rb
{% endhighlight %}

In another terminal run:

{% highlight bash %}
    ab -n 100000 -c 100  'http://localhost:4567/'
{% endhighlight %}

End enjoy your messages as they flow in the console, fairly load balanced in a round-robin 
among three EM instances. 

For extra fun, while ab is running, stop one worker and see how the load is distrubuted
among the remaining two. 

















