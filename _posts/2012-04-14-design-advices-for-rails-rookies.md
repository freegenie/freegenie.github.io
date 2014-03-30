---
layout: post
title: "Design advices for Rails rookies"
date: 2012-04-14 01:55
comments: true
tags: rails programming
category: blog
published: true
---

Sooner or later you'll find that your Rails application is becoming hard
manage. It does not matter how hard you try to keep things simple an DRY. 

If you plan to build and app which is
slightly more complex than a blog, then you'd better spend few minutes reading
this article. It's a collection of thoughts written by a guy
(me) who is trying hard (with good results) to keep a quite large code 
base easy to maintain. 

<!--more-->

Rails is an MVC framework for the web. Let's start giving a look at what those
three letters mean. I'm starting from the bottom just to keep the most
interesting part at last.

## Controller

This layer is where you app decided what to do with the input it receives from
the user. Controllers in MVC are responsible for user input validation
(validation? isn't it in the model?). We have been taught to keep controllers
as thin as possible, in order to keep our application maintainable, and this
is absolutely wise. The general teaching here is to keep much of the *logic*
away from the controller layer, moving it to the model layer.

Controllers in Rails are also responsible of how the data you are asking for
should be represented, at least for what concerns the *format* to be chosen. 

Controller layer is also where routing management resides. 

### before filters

Before filters form a chain a of method which are invoked in the order 
they are defined in your controller and in it's superclass.

While before filters are absolutely useful to keep your controllers 
readable, looking at the few lines of code you have in your controller action 
may lead you to the wrong assumption that you achieved that 'thin controller'
goal. Well, it may not be the case. Before filters are part of your controller 
action. If you define instance variables in your controller you are cheating
at keeping the controller thin.

## View

Views are responsible for data (state) representation. If you are using some
clear conventions on what-goes-where, it's very unlikely for you to get
in trouble here. This layer in Rails is very powerful and does a good job at
keeping complexity well organized.

The only risk here is to put too much logic into views. Again, we have been 
told to move logic away
from views, we have been provided with helpers to keep things clean. Helpers
are basically functions into modules, whose namespace reflects the controller
ones. (if you are using automated generators at least). 

Given that helpers are methods to be used in your templates, the only state
they can access is given by your template's instance variables. If you try to
do too much with your helpers, you'll very soon end up with a chain of smaller
or bigger functions, with more or less specialized responsibilities, that
basically do stuff with your instance vars. Consider that in a complex
template, made up of several nested partials, it becomes very easy to loose
track of the instance vars you define in your controller (and by controller I
also mean before\_filters). In such a wide scope, instance variables smell like
global variables.

## Model

Here is the interesting part. Model is the brain, heart and soul of you
application. At least this is what they say it should be. Very soon, while
trying to dodge the dangers described above, you'll understand why models have
this fame. For the rest of this article I'll refer to 'models' meaning just what 
Rails puts in app/models folder.

Model is the layer you usually start thinking to when planning your app. It's
where you store data, and where you guard for consistency of your database.
It's also the only part of the code which is not web-related at all. 
Unplug the web, your models are not going to die.

### Instance methods

In the effort to avoid fat controllers and helper madness you'll start to
enrich your models with new instance and class methods. 

As an example:

{% highlight ruby %}

class User 
  ...
  def full_name
    "#{first_name} #{last_name}"
  end
  ...
end

{% endhighlight %}

This is not that bad after all and can definitely have his meaning if methods like this are few. 
The only problem with this kind of stuff is that the model is worrying about a presentation of 
himself. A model can be represented in such a variety of different ways that relying on 
such methods for the purpose can make your rows count grow more than you think.

### Relations

Good thing about models is that they can define complex relations with 
other models, in the form of the well know has_many and has_one. What can go wrong here? 
In my experience, not much. The way ActiveRecord takes care of all the boring/repetitive aspects 
of SQL is simply terrific.

### Callbacks (and observers)

Callback is a chain of methods which are invoked during a 'save' action. This chain includes: 

- attribute validation
- all things to happen before the record is saved
- all things to happen after the record is saved

If you use them wisely, it's a good place where to organize your models interaction.

Validations should only be used when you want to provide a user with a soft failure, showing nice an
polite error messages. Validations should not be used as a mean of protection
for vital and possibly user-transparent attributes in your model. In fact they can 
be avoided by passing :validate => false to .save method.

I.e. you should not use them to protect your 'project_id' column if you are
never offering the user a choice to change it. It's better to use
'before_save' guards for such things.

Before\_validations should only be used when you want to sanitize some input
value or set a default for missing attributes. 

After save callbacks should only be used to change the state of other 
models when the current record is saved. This is actually the safer place where to do it, 
since after_save callbacks are wrapped inside the same transaction that saves the record. 

## Put your models on diet

What I've listed so far is all you need to put in your models, not a keystroke
more. Namely, don't use your models to: 

### Send emails 

Users' inbox is not your database. Furthermore observers and callbacks ready
to spit out emails to your thousands users while you are debugging and fixing
data in rials' console is something that makes me feel nervous so I tend to
isolate such communications in well defined places somewhere else in the
controller (domain objects, read later).

### Interact with external services

Generally speaking, there's nothing wrong with it and it's rather correct to think
about external services as something that belongs the model layer. I'd just
not use an ActiveRecord class for that. Use a Plain Old Ruby Object instead.

### Describe themselves in complex structures

I never used .to\_xml on a model in my life, and the few times I had to run
.to\_json it suddenly became a hell to customize. Complex json formatting should
go in the views, use something like [Jbuilder gem](http://rubygems.org/gems/jbuilder).

## So where does the fat go? 

In other classes. In few words, what Rails lacks of is at least a couple of 
more directories under 'app' folders, just to give newbies a hint that there's
something more to come when they'll start feeling masters of the views, models
and controllers folders.

Common practice is to put other collaborator classes into lib directory. This
is absolutely fine, but I would expect Rails, which is a strongly opinionated
framework, to take a position on this matter and not just leave it completely
uncovered. And by the way, I always think to the lib folder as the antechamber
for a plugin or gem extracted by your app.

Given that 'app' contains code that is strictly belonging to your application,
there are at least three other folders I usually add under 'app': 

- presenters 
- domain 
- mixins

### Presenters 

As described above, helpers lack of their own state, and views are as wild as
a jungle. Presenters help to contain all the logic which is required for one
or more model's representation. You can read more about presenters: 

- [here](http://robots.thoughtbot.com/post/13641910701/tidy-views-and-beyond-with-decorators)
- [and here](http://robots.thoughtbot.com/post/13641910701/tidy-views-and-beyond-with-decorators)

I prefer to craft my presenters by hand as Plain Old Ruby Objects.

### Domain

This is were I usually put all classes that are not ActiveRecord ones hold
domain logic as well. Classes in here usually use AR in composition.

### Mixins 

Depending on the patterns you follow, you may need this folder or not. 
This is the place where to put modules that you mean to share between
different classes.

## Conclusion

Maybe 'Big apps in Rails' would be a good title for a book. While we wait for 
that title to get published, all we can do is to help ourselves with our own
experience and keep posting our own practices like the article you just read.














