---
layout: post
title: Rvm rubygems
date: 2012-05-18 22:06
comments: true
category: blog
tag: rvm
---

[RVM](https://rvm.io/) is really cool, I've been using it for quite some time now and I can
say I'm a happy user. It really helps when you have many projects to maintain,
but it really shines when your projects start to become old. 

When managing a lot of client projects it not always possible to bump the
Rails version and keep the codebase up to date. Clients not always understand 
why this is to be done, regardless of your efforts to explain this need. 

<!--more-->

So it happens that you have to puts your hands in your own code you wrote
three years ago and, once you survive the sight of your own sins of the past, 
you have to face the hard thruth that you no longer have a local development 
version for it. 

This happened to me today, I had to startover, setting up everything from
scratch, with the only help of the (few) documentation I left to myself.

Unfortunately the project was so old that it was not yet Bundler enabled. 

Do you remember Rails 2.3 `rake gems:install` command? Well, it still works
fine but it does not bootstrap. The major problem with rake gems:install
command was his dependency on the Rails environment itself. If you don't have
the gems to make at least you Rails init process to complete succesfully, 
rake gems:install is almost useless. 

The project was still running in production, with it's own unix uid, so the
first thing I did was to run `gem list` on production env and to replicate
those on my local machine.

## rvm rubygems 

I had to bang my head for quite some time due to a strange error during
startup. I was not even able to run any rake command. The error was: 

{% highlight bash %}
"uninitialized constant ActiveSupport::Dependencies::Mutex"
{% endhighlight %}

After a short search I realized it was due to some rubygems incompatiblity. I
ahd to downgrade it on my laptop. Fortunately, rvm has a command for that: 


{% highlight bash %}
rvm rubygems 
{% endhighlight %}

You can specify whatever rubygems version you want to use. It downloads and
install. In few seconds I was back up and running. I only had to struggle a
little bit more to understand how to restore the prior verison. There's a
command missing in the doc, which is:

{% highlight bash %}
rvm rubygems current
{% endhighlight %}

## Conclusion

I heard of a lot of people being happy with [rbenv](https://github.com/sstephenson/rbenv). 
I did not try it yet, maybe I'll give a look at it someday, even though for
now I'm really happy with RVM and I'd highly suggest it for managing different
versions of ruby.




