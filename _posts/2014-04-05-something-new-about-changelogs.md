---
layout: post
published: true
title: Something new about changelogs
---

Lately I realized a thing which made me sad: changelog formats are far from standardized. 

I stumbled upon this thought while thinking about a way to have `bundle
outdated` to print out a kind of changelog excerpt to help me get an idea
on whether I had to update a gem or skip the latest version for this time. 

The first step to achive this would be to have a kind of changelog parser to 
extract machine readable data. So I started to look around for a
changelog parser. I hardly found something, which somehow surprised me. 

## The official CHANGELOG format

I wasn't able to find an **official spec** for changelogs. I found instead
a [page hosted on gnu.org](http://www.gnu.org/prep/standards/html_node/Style-of-Change-Logs.html)
which lists few advices on how to create a changelog file. It's venerably
old. It describes few rules expecially suited for programs written it C
language. 

It lacks of several directions I was expecting to find. For instance it doesn't
say anything about version numbers. It's all about dates of changes. 

## Different formats and conventions

I decided to have a deeper look at several popular OSS projects to see how they
behave. I was expecting to find a changelog of any format in the root
directory of the project. I was almost never the case. 

### The Linux Kernel 

The linux kernel does not have a changelog file in the root directory. There's
a Documentation folder, [with a "Changes" file
inside](https://git.kernel.org/cgit/linux/kernel/git/stable/linux-stable.git/tree/Documentation/Changes?id=refs/tags/v3.13.9).  It's a documentation
file, not a changelog strictly speaking, so it does not track changes in specific versions. 
Other changelog files are present in specific component folders. 

### Ruby on Rails 

There's not a changelog in the main folder, but [several "CHANGELOG.md"](https://github.com/rails/rails) 
files inside each component folder. It's a markdown file, with no version numbers
and no dates. 

### WordPress 

[No changelog at all.](http://core.svn.wordpress.org/trunk/)

### Django 

[No changelog.](https://github.com/django/django)

### Rubygems.org

An [History.txt file][2] with version numbers and no dates.

## What about libraries?

Since I'm a Ruby guy, let's have a look at some popular Rubygem: 

### Sidekiq

Sidekiq has a [Changes.md](https://github.com/mperham/sidekiq/blob/master/Changes.md) 
file which has version number and a bullet list item for each change. No dates. 
It has code examples.

### Nokogiri 

Has a [Changelog.rdoc](https://github.com/sparklemotion/nokogiri/blob/master/CHANGELOG.rdoc) 
file with version and dates as headings, and bullet lists for features and bugfixes.

### Devise

Has a
[CHANGELOG.md](https://github.com/plataformatec/devise/blob/master/CHANGELOG.md)
with version numbers and changes in a bullet list, just like Sidekiq. 

## Do I need a changelog? 

There are times in which it doesn't make sense to write a changelog. 
A small team working on a private project may not find any benefit
in writing a changelog. But if you work on an opensource project, expecially
on a library, you should definitely have one. 

Dependencies on a project tend to become difficult to manage also because you
don't have a clear view of what changed. 
[Semantic versioning][1] helps us a lot
to quickly understand if it's safe update a component or not. 

Is it a bugfix?
You should update ASAP. Is it a minor change? It's safe to update and you
should, because you want to stay close to the edge, right?
Is it a major change? You should take the risk to update because the world is moving on. 
_But it's not always that easy._

Even if it's a bugfix, I want to know what changed. And
because every time I run a `bundle outdated` it's always a long list showing
up, I would have a quick summary. 

Many times, even minor version changes are too easily applied. If a change is 
backward compatible you shuold still **ask yourself if that change is something 
you would benefit of**. Most of the times it's a new feature introduced, which 
you may not need. Moreover a backward compatible change may introduce a bug. 

Developers are often caught by a kind of **up-to-date fever**. I think part of
it is due to the fact that it's hard to have a clear view of your
**outofdateness**. 

## Why a shared format is important

Going back to the inital thought which led to all this typing, a shared
standard for changelogs would make it possible to have machines do something
interesting with those files. 

[Bundler][3] is one of the many dependency management which may benefit of a
parsable format. Even an online service would be possibile with little effort,
to send you a weekly digest of the changelogs of your project's dependencies. 

## Modern requirements for a changelog

Let's have a look at what the specs for a modern changelog should be. 

### Version numbers 

The most important thing a changelog should tell is the version number the
changes refer to. Listing dates is much less important. 

### Markdown 

Having a look at what most projects are doing, I think any kind of 
text friendly markup language is a desirable feature.
TXT is great, but the world is on the web and
with a tiny markdown effort we can make web pages much more pleasant to read. 

Markdown would also make possible to have automated syntax highlight for code
samples and all the stuff we already know and appreciate. This should be optional though. 

### List of changes

Those should be listed from the most recent to the oldest. Each item should
start with an item list marker (this may change depending on the makrup
language adopted). Each item should be composed of at least a short summary. 

Optionally, a change may have a longer description of the change, links, code
example or any other thing a developer may want to add. 

### Authors

This is far less important than the previous three. I would hardly make a
decision on whether a change is something I would apply or not based on who
did that. That should be something you dig into the CVS when interested in. 
This one too should be optional. 

### Dates 

I don't know what to do with dates in a changelog. Again, they may have a
meaning, but what I'm trying to come up with is a format which includes the
basic facts a developer may need to decide if a change is worth upgrading or
not. I don't think anybody would take this decision based on a date. I
consider the date to be optional too. 

It's worth nothing that if you like to put dates in a changelog, you may want
to put the dates on change items *and* on version numbers. If you are a date
maniac, you'd probably want to know how many time passed since the last change
and the version release.

### Example 

I think something like this should would go as a standard changelog: 

    # Acme Products 

    ## 2.1.2 (2002-03-10) 

    * Added a button to fire missles

      You shuold used this only if in danger. Don't fire for offense. Ever. 

      Author: Fabrizio Regini <freegenie@gmail.com> 

    * Added breakes

    * Fix a bug in cosmic ray controller

Which in markdown translates into this:

![formatted changelog example](/images/formatted-changelog-example.png)

## Conclusion 

I think that having a shared format for Changelogs is a little bit an
**utopia**. But the possibility to have your changes show up nicely 
into one of the many dependency management softwares out there may be a huge
stimulus for developers to agree on something. 

**What do you think?**

[1]: http://semver.org/ "Semver.org"
[2]: https://github.com/rubygems/rubygems/blob/master/History.txt 
[3]: http://bundler.io/






