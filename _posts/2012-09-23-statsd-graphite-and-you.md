---
layout: post
title: Statsd, Graphite and you
date: 2012-09-23
comments: true
categories: blog
description: How to start using Statsd and Graphite the right way and quickly
published: true
tags: statsd graphite lean
---

![my first dashboard](/images/statsd-dashboard.png)

Measuring everything is crucial to get into a sane build/measure/learn cycle. But more importantly one should be able to measure things quickly when the need arises, with minimal time and cost effort. 

If you've read ["The Lean Startup"](http://www.amazon.it/The-Lean-Startup-Entrepreneurs-Continuous/dp/0307887898) or similar books like I've done recently, you know what I mean. 

One of the most efficient solutions these days seems to be [Statsd](https://github.com/etsy/statsd) in combination with [Graphite](http://graphite.wikidot.com). This setup became popular because it's the one used by Etsy's team, who very generously posted his experience in this [famous post](http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/)

So I started digging into it, trying to read as much as I could setting up the required environment. [Thanks to this guide](http://geek.michaelgrace.org/2011/09/how-to-install-graphite-on-ubuntu/) it was pretty straightforward. This post is an attempt to summarize what I would I knew before starting but I did not find around. 

## You can go with the default Graphite installation

While reading around, I thought most of the job was done by Graphite being Statsd just a UPD bridge to it. It was interesting to learn a little bit more about Graphite internals, but it's really not required to get up and running quickly. My poor understanding of Statsd led me think there was something I had to configure on the Graphite side in order to get the result I wanted. 

Reading Statsd source code helped, and I realized the following. 

## Statsd does much in few lines of code

Statsd is a NodeJS deamon which listens for the data you send him and **does things** to the data before sending it to Graphite. 

To understand what it does to the data, it's important to understand the interface Statsd defines. There are basically three different flavors of measurements you can send: 

1. timers
2. counters 
3. gauges

Statsd keeps data in memory and flushes to Graphite at regular intervals. **The default flush interval is 10 seconds.**

### Timers 

Timers give you the possibility to track the amount of time taken for a certain operation to compute. Timers are great for instance if you want to track the time taken for a page to load. In the [ruby driver](https://github.com/github/statsd-ruby/blob/master/lib/statsd.rb) timers can be sent using a block by the `time` method, or just using the `timing` method.

When flushing timers to the Graphite backend, Statsd computes and sends the following stats:

1. count
2. lower
3. mean
4. mean\_90
5. std
6. sum
7. sum\_90
8. upper
9. upper\_90

The meaning is clear. Sending just one timer instruction you'll find in Graphite all you need to get the true meaning behind the numbers. A page load may be fast in the average, but you may notice that _sometimes_ it's tremendously slow by having a look a the *upper* stats. 

### Counters

When you send a count instruction, the value is kept in memory and flushed to Graphite at the given interval. **Subsequent counters for the same key increase the value.** This means that if you have three processes each one sending a count of 20 to Statsd, what you will see in Graphite is 60. This number will keep growing on subsequent calls, unless you send negative values, in which case the counter is decremented.

Counters are supposed to track stuff like the number of user signups (if you send 1 each time a new user is created) or number of clicks on a given link. 

So to recap, using the ruby driver, you have `count`, `increment` and `decrement`.

### Gauges

**Gauges are for monitoring**. If you are setting up your own monitoring script to run in a cronjob, you'd want to use the `gauge` method. The difference between counters and gauges is that, on subsequent calls, the value is not increased. What statsd will send to Graphite is the last value stored in the gauge key. 

So if you send 20 for three times, and then 10 the next minute, in Graphite you'll have two data points, one at 20 and one at 10. 

I'm using two gauges to monitor the number of lines in Nginx error and access logs. The idea is that if the error log grows too much there's something wrong, for instance with Nginx not being able to pass the request to Unicorn.

I have a cron script that simply counts the lines every 30 minutes, simple and effective. 

### Debugging

One thing that really helped me to understand what was going on was to run Statsd in debug mode. You'll have interesting output on the console each time data at each flush interval.

## Finally 

I just started to scratch the surface of Statsd and Graphite. It is so easy to start tracking stuff that the limit is your imagination. I was impressed by how much clearer things become when you can put together two different charts, side by side, which apparently do not have much in common. Suddenly, data tells the story. You get an extreme focus on the problem, and yet you can zoom out and read other pieces of the puzzle with few clicks. 
It's definitely something that should be there with your project from the beginning. 
