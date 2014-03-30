---
layout: post
title: Backup Gridfs files as normal files
published: true
category: blog
description: This is the monster I created to get gridfs files backed up as if they where on filesystem 
---

This is the monster I created to get gridfs files backed up as if they where on filesystem: 

<http://gist.github.com/534099>

One thing I wanted  was to keep the script in one file, and to make it work with the least dependencies possibile. 

This is what it does with the dump commnad 

- dump all collections but fs.chunks as usual, with mongodump 
- Gzip all bson files
- create a directory for grids files
- dump gridfs files to filesystem via Ruby, only if id+checksum did not change
- remove fs files that are no longer in the database

On restore 

- unzip bson files
- restore the database from bson files
- iterate over fs.files collection, read dumped file from filesystem and restore it into gridfs.

The operations above can be performed on a subset of databases. 

I learned a couple of things working on this script: 

- What :snapshot => true is for. I ended up in an endless loop on restore. Cursor was beeing fed with new records on every iteration.
- Authentication in mongodb is painful. You have to add admin user to each and every database in order to make this script work for the whole instance when --auth is on.  

Enjoy!