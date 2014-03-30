---
layout: post
title: Happy Provisioning with Chef
date: 2012-07-15
comments: true
categories: blog
description: Writing about how I started working with Opscode's Chef
published: true
tags: chef provisioning
---

Few days ago I started a system administration activity 
which involved, among the other things, to move several services we had hosted on a local data center to cloud hosted VMs.

I approached this activity making a short plan, and started configuring machines and doing all
the stuff manually. I was quite confident that it would not cost me much, given I have full knowledge 
of all the services hosted there. It was either Wordpress or Textpattern instances, and several Rails driven 
website and web applications. 

We never had the time and the will to start using any kind of automatic provisioning tool. I don't know exactly why, maybe we were scared by the learning time required. Come on, after all you only have to `apt-get install` it! Well, now I know you don't realize how much you need it until you have to replicate everything somewhere else. 

The second day I started to realize I had to take that chance to start using a provisioning tool. 
Even though I had clear in my mind all the necessary steps to take in order to move one thing from one place to the other, doing all configurations manually is simply something human brain in not made to handle, 
especially if you jump from one task to another trying to save time.

The final blow was given by a particular package one application was requiring. We did a manual compilation and the procedure was not documented anywhere. It took me half a day to sort it out. Totally energy draining activity. It never had to happen again. 

## Chef or Puppet

I did a short research, knowing that something called [Puppet](http://projects.puppetlabs.com/projects/puppet) was out there. At the same time I knew there was something called [Chef](http://www.opscode.com/chef/) doing same thing. I'm a Ruby developer, but I know Python as well, having worked with it for a few years in the past. It should not be so, but knowing you master the programming language a tool is built on makes you more confident about it. If something is not clear or lacking docs you can always inspect the code and understand it. 

If fought hard with myself to keep this feeling down in the hole, but I admit it played a role in the final decision. Chef seems to be well documented and tremendously community supported, but not as much as Puppet, being that a much more established and aged project. 

I played a little with Chef, I was able to get up and running very quickly, I didn't feel like I could achieve the same speed with Puppet given my confidence with ruby files, so the choice was done. (Hours later this choice was done, I was really happy I made it)

## Chef Solo 

Chef Solo is the version of Chef which doesn't require a separate server instance to run. It's basically just a gem with an executable that runs your scripts. Start with it of course. In my case I have a folder with cookbooks, data bags, and node files and I just rsync everything to the node I'm working on. 

## Time saving hints

I suggest you to go and watch [this RailsCasts](http://railscasts.com/episodes/339-chef-solo-basics?view=asciicast). You'll have a nice hands-on short introduction. Next, these are a few time saving hints I wish I was given before starting. 

### File names matter 

Chef expects to have file names matching other file names or directory names sometime. This is important to understand, especially in the case of Data Bags: DataBag files contain json configuration options, which *must* include an "id" key, and the "id" key has to match the file name. 

### Read the documentation page about 'Resource'

[This page](http://wiki.opscode.com/display/chef/Resources). You'll find there most of the DSL commands you need to know early, such as create users, create groups, download files, install packages, and run bash scripts.

### Node configuration options precedence 

There's a precedence system in setting recipe options. You can have them in `attributes` folder, you can have them in `node` files, and maybe somewhere else I still don't know. You can have a `default` value, you can `set` one or you can `override`. 

### Encrypted Data Bags are not ready for Chef Solo 

Encrypted Data Bags are the feature which allows you to keep encrypted configuration keys saved into your provisioning files, and have them decrypted during installation, so you don't have to copy and paste ssh keys for example, or type the root mysql password each time during installation. This is the feature I was more interested in, but unfortunately I had to fight a little to make it work with Chef Solo. But in the end I did it, I'll post shortly on it, just the time to clean it up a little.

## Conclusion

Folder structure is intuitive, I found a lot of ready made recipes and cookbooks I'll need to use, like the one for user specific RVM installation. I like the result I achieved with just one day with it and very few lines of ruby code. Sounds like a good start. 




