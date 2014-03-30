---
layout: post
title: Ruby Inject
category: blog
description: 
---

The best reminder I can think about for Ruby inject method is this:

{% highlight ruby %}
(0..100).inject {|b,v| b + v} 
=> 5050 
{% endhighlight %}

It has got to do with a the story I heard about "Carl Friedrich Gauss":http://it.wikipedia.org/wiki/Carl_Friedrich_Gauss. 
When he was a kid at school the teacher gave the class a task to compute the sum of numbers from 1 to 100. He did not use
irb obviously, nonetheless he was able to answer in very little time. 

He noticed that the sum of number placed at the same distance in the row was always 101, like this: 

{% highlight ruby %}
1 + 100 = 101 
2 + 99 = 101 
3 + 98 = 101 
{% endhighlight %}

And given that couples are obviously 50, the answer is simple: 5050! What if he had a laptop! 

Another thing that helps me to remind what's for is to call it *reduce*, which is actually an alias for it in Ruby. 




