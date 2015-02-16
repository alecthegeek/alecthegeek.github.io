---
layout: post
title: "Handy Hack:Windows Beyond Compare 3 on OS X"
date: 2013-05-26 18:15
comments: true
categories: [OS X, Git]
---

My colleagues at work, many of who work on Windows, swear blind by Scooter Software's [Beyond Compare](http://www.scootersoftware.com/moreinfo.php).
They even claim that it is so good they don't want to use Mac for development without it (there is a version for Linux).

So I present a wrapper script that enables beyond compare to run under [Wine](http://www.winehq.org/)
and used as the difftool in my Git installation.

1. Install Wine via [Homebrew](http://mxcl.github.io/homebrew/)
2. Download the Beyond Compare installer and invoke using the Wine program
3. Install the following script in your PATH (I use `~/bin`)
``` bash
#! /usr/bin/env bash

while [[ -n $1 ]] ; do
   if [[ $1 == -* ]] ; then
     OPTS="$OPTS \"$1\""  # Just an option
   else
     OPTS="$OPTS \"$(echo "$1"|sed -E -e 's@^([^/])@'$PWD'/\1@' -e 's@^/@@'  -e 's@/+@\\@g')\""
   fi
   shift
done

/usr/local/bin/wine 2> /dev/null ~/.wine/drive_c/Program\ Files/Beyond\ Compare\ 3/BCompare.exe $OPTS
```

You can then set the wrapper script to be your compare tool in the various tools you use.
I've tested this using `git difftool`, please let me know if you have problems.

The underlying assumption is that
Wine provides a drive that maps to the root of your file system (on my system it's `z:`)
:nd that work is done relative to that
