---
layout: post
title: How to tar a file via ssh when space is over
description: How to tar a file via ssh when space is over
date: 2012-09-25
comments: true
categories: blog
published: true
tags: ssh tar unix shell tips
---

It may happen that the filesystem gets bloated by a growing log file when some process gets mad 
and starts writing to logs in a blind fury. Or maybe you forgot to add that server to nagios for monitoring of free space.

Whatever the reason, you may need to delete a big file, but you don't want to just delete it. Maybe you want to 
free system space to allow services to restart (ASAP) but want to keep the big file for analysis. 

With `tar` you can stream over ssh and have the tar saved on the other end. 

Command looks something like this: 

    tar cz - offending-big-file.log | ssh you@other-server " cat > offending-big-file.log.gz"

Remember to restart or kill any process holding a reference to the deleted file, otherwise the system won't free space on disk.