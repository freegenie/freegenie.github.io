---
layout: post
title:  Change background when sun goes down
date: 2012-04-12 22:27
comments: true
tags: productivity vim
category: blog

---

I love to edit in Vim on a beautiful dark background, but lately I had to face
the hard truth. If you don't have an antiglare screen and if your desktop is
kissed by sun the whole day, you simply can't use it for daily work unless you
accept to see your own image on the screen at every single keystroke.

I never liked light background themes in terminals, and I still don't, but I
have to admit this [Solarized](http://ethanschoonover.com/solarized) theme is really great. I slowly started to switch
to it for daily job and I can say my life changed in better. 

<!--more-->

But the first love never dies, so I find myself now and then changing the
[iTerm2](http://www.iterm2.com/#/section/home) and Vim theme to come back to a softer and more inspiring dark screen. 

iTerm2 has plenty of features, so the easiest way I found to get quickly a new
tab in dark is to create a new profile and make a shortcut for it. 

![iterm2-shortcut](/images/iterm2-shortcut.png)

The only thing you have to remember is to use the shortcut each time a new tab is
required. 

For vim, well, [this page](http://ethanschoonover.com/solarized/vim-colors-solarized) shows how to
setup a shortcut with F5 to switch among light and dark background. 
I don't find it very useful. I open and close vim frequently. 
Using the shortcut to change background after the first flash of light
background hit your eyes is annoying, but fortunately you can avoid that with 
just one line of vim scripting in your vimrc: 

{% highlight vim %}
if strftime("%H") > 19
    set background=dark
else
    set background=light
end
{% endhighlight %}

Use dark background after 19, which is the time I'm usually commuting.

All seems fine so far, I'll let you know how it goes!

