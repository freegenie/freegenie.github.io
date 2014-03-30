---
layout: post
title: Understanding Git, merge and rebase
category: blog
description: in which I explain what I understood so far of Git merge and rebase features.
---

[vjt](http://sindro.me) has been recently mentoring me on [Git](http://git-scm.com/), which is my new favourite 
[SCM](http://www3.us.kernel.org/pub/software/scm/git-core/docs/v1.4.4.4/glossary.html#ref_SCM). 

I was previously sticking around with Subverison, which is another good SCM, at least as long as you need an SCM
just to checkout, commit, and maybe revert something to a previous version. Running multiple branches has never 
been easy, and it always requires a server side 'something' which is not always as fast as it should be. 
That led me many times to rather choose not to branch something when it was actually the right thing to do. 
My life changed when I started to use Git, which is the reason why I decided to honor his awesomeness dedicating
this short article to it. 

I'm not going to analyze and Git in deep, I would just focus on a couple of key features I understood relatively 
too late. If I had those two things clear in mind from the beginning, I won't probably be asking silly questions
all around for so long. 

To fully understand Git, it's important to understand (or recall) how diff and merge tools work. 

## Diff and Patch

From the dawn of time in unix programming people has been passing around pieces of code via the diff tool, 
which is basically a program that compares two text files and generates a formatted representation of the
differences between them. With diff you can tell people that they are misspelling a variable name in a thousand
lines long file, in an unequivocal way. The output of the diff format is so unequivocal, that you can use that
as input for another famous tool in source code management: *patch*. 

What the *patch* command does is to take a diff in input, and apply that to your source code. After the *patch* 
command runs, you end up with your code changed as described by the diff. 

A quick example. This is going to create three files: 

{% highlight ruby %}

    echo "This is a file with a line of code" > a.txt 
    echo "This is the first line\nthis is the second line" > b.txt
    diff a.txt b.txt > a-b.diff

{% endhighlight %}

The a-b.diff file contains: 

{% highlight diff %}

    1c1,2
    < This is a file with a line of code
    ---
    > This is the first line
    > this is the second line

{% endhighlight %}

Now, run the *patch* command on file a.txt with, instructing it with the diff: 

{% highlight bash %}

    patch a.txt a-b.diff 

{% endhighlight %}

You'll have the a.txt file *patched*, which means automatically fixed to match the b.txt file's content. 
The *diff* format is so unequivocal, that the *patch* command can do smart stuff with that, for example 
to inform you that the destination file is not in the state the author of the diff was expecting it to be. 

Try for example to run the patch command a second time: 

{% highlight bash %}

    patch a.txt a-b.diff 

{% endhighlight %}

You'll get a message like this: 

{% highlight bash %}

    patching file a.txt
    Reversed (or previously applied) patch detected!  Assume -R? 

{% endhighlight %}

Useful to prevent you to mess around with many different diff files. Well, on a larger scale, this is an important
part of the daily job of a Git repo instance. Managing diffs and patching files. To "apply" a commit on something, 
basically means to apply a patch. 

## Merge 

We saw that the *patch* command can get quite verbose and interactive when the file you are patching is not 
what it was supposed to be when the diff was created. If we where to manage source code on a large team only 
with the help of the diff tool, we won't probably go far. Things can get out of sync really fast in real life.
This is why the clever people in the 70s invented the *merge* tool. 

A quick look at the merge man page can say more than anything I can write about it. What it basicaly does is: 

1. take file one
2. take file two 
3. try to understand how file one could have become file two 
4. try to apply the same changes to file three

As you can see this is much different from appling a simple patch. Merge does not do any assumption
on the destination file. It can be whatever. As long as it can apply the canges 'cleanly' it's happy and 
will exit silently. Otherwise, it will output a 'conflict' on the destination file.

One thing to understand here is that, if the destination file is the same as one of the source files, merge
won't produce any output, the change will always be successful. 

Let's try it with an example. Use 'merge -p' to print the result instead to write the destination file: 

{% highlight bash %}

    echo "This is a file" > aa.txt 
    echo "This is another file" > bb.txt 
    merge -p aa.txt bb.txt aa.txt 
    merge -p aa.txt aa.txt bb.txt 
    merge -p bb.txt aa.txt bb.txt 
    merge -p bb.txt bb.txt aa.txt

{% endhighlight %}

Things change when the destination file has changes. You can try it with another example:

{% highlight bash %}

    echo "This file is called 'C'" > cc.txt
    echo "This is a file" > aa.txt 
    echo "This is a line\nThis is another line" > bb.txt 
    merge -p cc.txt aa.txt bb.txt 

{% endhighlight %}

You'll get the following conflict: 

{% highlight diff %}

    echo "This file is called 'C'" > cc.txt
    <<<<<<< cc.txt
    This file is called 'C'
    =======
    This is a line
    This is another line
    >>>>>>> bb.txt
    merge: warning: conflicts during merge

{% endhighlight %}

The conflict shows two different 'versions' of the same blocks of text. Those are the parts of the 
destination file that where different from both aa.txt and bb.txt. Merge can't decide which one to 
keep, and passes the ball to you. All you have to do now is to open the destination file and edit 
the conflicting parts. 

## Rebase, cherry-pick, merge

Once you master these concepts, it won't be hard to understand how to boost your productivity and 
handle your source code with confidence even in the most complex situations. 

*Rebasing* A to B basically means to apply *all* the commits that A needs to reach the status of B. 
As we saw, it means to patch the code diff by diff in the order the commits appear on the track from A to B.

*Cherry-picking* a commit from branch A to branch B means to patch the code with that very specific commit, 
and nothing else, very useful when porting bugfixes from a branch to another.

When you want to join two different development branches, and keep track of the branches history in 
the repo, use merge. Say you want to *merge* branch B on branch A, you will end up with a 
'merge commit' on branch A that changes the files on branch A according to the status of branch B. 



