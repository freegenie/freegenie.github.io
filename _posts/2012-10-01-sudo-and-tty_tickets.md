---
layout: post
title: Sudo and tty_tickets option
description: How to keep sudo sessions open across ssh logins
date: 2012-10-01
comments: true
categories: blog
published: true
tags: sysadmin sudo ssh
---

It's about ssh and sudo sessions. Working on a new 12.04 instance, I noticed 
it was not keeping my sudo sessions open across ssh connections. 

Such behaviour was a default in previous releases. I never had to configure it 
and it just worked. It's quite annoying to input your password repeatedly 
when you are running a bounch of automated script to configure a new instance, 
expecially if you are testing those configurations and you run them on and on.

So here is the line you have to add to your `/etc/sudoers` to do the trick: 

	Defaults 		!tty_tickets

I've read the docs too late once again. You please go and read the docs to know
how it works. 

## Security? 

Keep in mind that changing this configuration will allow anyone who is able to 
access your shell on the remote machine to run a sudo command without password
prompt. 