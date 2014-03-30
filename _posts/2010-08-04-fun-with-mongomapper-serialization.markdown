---
layout: post
title: Fun with mongomapper serialization
published: true 
category: blog
description: "One cool feature of mongomapper I understood recently is automatic object serialization. What it does is to allow developers to write code to map their own objects into a mongodb database. 
It works in a very simple way: every object you want to save to mongodb has to implement a very simple interface, like this"
---

<em>This post covers mongo_mapper v 0.8.2</em>

One cool feature of mongomapper I understood recently is automatic object serialization. What it does is to allow developers to write code to map their own objects into a mongodb database. 
It works in a very simple way: every object you want to save to mongodb has to implement a very simple interface, like this:


{% highlight ruby %}

class MyOwnDatatype 
  def self.to_mongo(value)
  ...  
  end

  def self.from_mongo(value)
  ...
  end
end
  
{% endhighlight %}

Mongomapper by default extends some basic ruby data types to make them compliant to this interface. 
[You can find them here](http://github.com/jnunemaker/mongomapper/tree/v0.8.2/lib/mongo_mapper/extensions)

The default data types are just the basic stuff, the kind of stuff a ruby developer expects to be able to save in any database. Extending someone of those datatypes, you can overcome little problems in an elegant way. Imagine you have the following model: 


{% highlight ruby %}

class BlogPost 
  include MongoMapper::Document 
  key :title, String
  key :content, String 
  key :tags, Array 
end

{% endhighlight %}

A classic blog post model with tags as an array in the document itself. It works fine, but in my controller I want to be able to pass tags as a comma separated string, like in the following example:

{% highlight ruby %}

>> b = BlogPost.new(:title => 'a post',  :content => 'the post', 
  :tags => 'music,sport')
>> b.save  
>> b.tags 
=> ["music,sport"] 

{% endhighlight %} 

As you can see, we got an array with the full string passed. 
Coming from a lot of ActiveRecord coding, I thought to implement a before_validation callback at first (yes, we have them in mongomapper too). But that's the hard way. The easy way is to subclass Array into a SuperTagsArray:

{% highlight ruby linenos %}

class SuperTagsArray < Array
  
  def self.to_mongo(value)
    _new_value = []
    value.each do |array_element|
      array_element.split(',').each do |sub_item|
        _new_value << sub_item 
      end
    end
    _new_value
  end

  def self.from_mongo(value)
    value || []
  end
end

{% endhighlight %} 

and in our model we can update the tags mapped object into: 

{% highlight ruby %}

class BlogPost 
  include MongoMapper::Document 
  key :title, String
  key :content, String 
  key :tags, SuperTagsArray 
end

{% endhighlight %} 

From now on, we can save tags as either an array or a string:

{% highlight ruby %}

>> b = BlogPost.new(:title => 'a post', :content => 'the post', :tags => 'music,sport')
=> #<BlogPost title: "a post", _id: BSON::ObjectID('4c606a7b069cec0a05000003'), tags: ["music", "sport"], content: "the post">
>> b.tags
=> ["music", "sport"]
>> b.tags = ['rock', 'pop']
=> ["rock", "pop"]
>> b.tags = 'fun,serialization'
=> "fun,serialization"
>> b.save
=> true
>> b.tags
=> ["fun", "serialization"]

{% endhighlight %}

The end. Thanks for reading. 

## Some related slides 

Ab interesting presentation of mongomapper, which also talks about this feature: <http://www.slideshare.net/jnunemaker/mongomapper-mapping-ruby-to-and-from-mongo>

