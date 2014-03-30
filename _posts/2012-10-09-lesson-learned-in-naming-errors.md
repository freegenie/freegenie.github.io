---
layout: post
title: Lesson learned in naming errors
description: 
date: 2012-10-09
comments: true
categories: blog
published: true
tags: mistakes
---

Today I've been working on a legacy app of mine for a reported bug. This was
an old application I did several months ago, more than a year. 

The reported error was cryptic, something like "Error at rows 1,2,3,33,44 in
field 'foo' ". The app parses an Excel file and performs some operations based
on it's content.

For various reasons, I didn't have the development version on my machine
anymore due to a system recovery I had to take since then. 

During previous support operations, issues originated from strange input chars
inside the Excel file, so I had a little bias toward the kind of fix required.

Long story short, it took me several hours to setup my development env again,
and, once I got it, I was able to give the requested support to the
client. Just to explain him that **there wasn't actaully any problem in the
software**. The client had to wait hours just because the error message 
was not explaining the error properly. 

It turned out that the problem was with some violated rules in the
Excel file format. That violation was hard to spot after months, **i forgot
the logic, and the client forgot it as well**. 

What would have helped is a clearer message, something like: 

    There is problem at lines 1,2,3,33,44. Please check field 'X'. 
    This error is the result of a validity check on field 'X' against
    field 'Y'. Values of 'X' must correspond to 'Y' in the whole file. 

Some additional hint for me and him too would have avoided the need to even 
open back the code to read from it. 

## Conclusion

Always take care of error messages you present to the user. Ask yourself: 
"what will the user X understand from this message?", "what will I understand
from this message after one year?". You may save time to the client and
headackes to yourself.


