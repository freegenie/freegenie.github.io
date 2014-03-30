---
layout: post
title: A bundle update story 
date: 2012-06-16 22:06
comments: true
tags:  bundler foo
category: blog
---

I've tried to upgrade a Rails project from version 3.0 to 3.1 (in first instance, with the idea to move then to 3.2). I had a problem with bundle that gave me the opportunity to dig a little bit more inside of it. Sometimes you can't just `bundle update` to resolve conflicting gems, especially during the attempt to upgrade an aged codebase.

<!--more -->

## The starting point

The first thing I normally do in such case is to change Gemfile to explicitly ask for the new rails version. In this case version 3.1.6 (1). Doing this and running `bundle udpate rails` I get the following error: 

{% highlight bash %}

     deploy (rails31) ✗ bundle update rails  
    Fetching gem metadata from http://rubygems.org/.......
    Fetching gem metadata from http://rubygems.org/..
    Bundler could not find compatible versions for gem "multi_json":
      In snapshot (Gemfile.lock):
        multi_json (1.3.2)

      In Gemfile:
        rails (= 3.1.6) ruby depends on
          multi_json (< 1.3, >= 1.0) ruby

{% endhighlight %}

Running `bundle update` will rebuild your snapshot from scratch, using only
the gems in your Gemfile, which may resolve the conflict.

The problem in this particular case was multi_json gem. In Rails 3.1.6, it's dependency is set to be greater that 1.0 but lower than 1.3, and my codebase seems to be using 1.3.2 for some reason. 

Given that my Gemfile is not referencing `multi_json` (I actually didn't even
know what it was) it's clear that some other gem I'm using once asked for it. 

## A look at Gemfile.lock

Gemfile.lock is extremely clear to read when doing such investigation. 

Looking for `multi_json` there here's what I find: 

{% highlight bash %}

    mongo (1.3.1)
      bson (>= 1.3.1)
    multi_json (1.3.2)         <------------
    multipart-post (1.1.2)
    mynyml-redgreen (0.7.1)

{% endhighlight %}

multi_json at root level, version 1.3.2. This means that my code base at
present is making use of this precise version.

{% highlight bash %}

    oauth2 (0.4.1)
      faraday (~> 0.6.1)
      multi_json (>= 0.0.5)   <-----------

{% endhighlight %}

multi_json is required by oauth2 gem version 0.4.1. This gem is only asking
for multi_json to be higher than 0.0.5. 

{% highlight bash %}

    selenium-webdriver (2.21.0)
      childprocess (>= 0.2.5)
      ffi (~> 1.0)
      libwebsocket (~> 0.1.3)
      multi_json (~> 1.0)     <------------
      rubyzip

{% endhighlight %}

multi_json is being required by selenium-webdriver. This time the minimum
required version is 1.0.

{% highlight bash %}

    simplecov (0.6.1)
      multi_json (~> 1.0)   <------------
      simplecov-html (~> 0.5.3)

{% endhighlight %}

simplecov (the beloved one) is asking for multi_json ~> 1.0 too. 

Seems like everybody is loving loving multi_json. Luckily there seems to be no big incompatibility with Rails 3.1, since we can just move it a version, say ~> 1.2.0 and everybody should be happy. 

## Downgrading multi_json to ~> 1.2.0

The problem is: how to do that? How do I tell bundler that once I was happy
with version 1.3.2 of a given dependency but now I have to downgrade it?

Since this gem is not in my Gemfile I have no way to specify an exact
version for it. At least until you include it in your Gemfile. 
It sounds strange to have a line in Gemfile for a dependency of your
dependencies, but this is the only way I found to force Gemfile.lock to
update. Once the job is done we could safely remove it.

I reverted to rails 3.0.10 first and added this line: 

{% highlight ruby %}

    gem 'multi_json', '~> 1.2.0'

{% endhighlight %}


Running bundle, here we are: 

{% highlight bash %}

     deploy (rails31) ✗ bundle 
    Fetching gem metadata from http://rubygems.org/.......
    Fetching gem metadata from http://rubygems.org/..
    You have requested:
      multi_json ~> 1.2.0

    The bundle currently has multi_json locked at 1.3.6.
    Try running `bundle update multi_json`

{% endhighlight %}

And after a `bundle update multi_json`, looking at Gemfile.lock we are finally
running with version 1.2.0. 

Now we can upgrade rails to 3.1.6 and no more issues.

## Off the camera: the spermy operator

Something that made waste some time during this issue was my poor knowledge of
the spermy operator ~>. I was initially convinced that typing ~> 1.2 was the
way to go, but I was getting the gem updated to 1.3.6 nonetheless.

This is because the spermy operator only allows the *last* part of your
version number to change, while it *locks* all the rest. So ~> 1.2 locks '1'
and allows '2' to change, while ~> 1.2.0 locks '1.2' while it allows '0' to change. 




