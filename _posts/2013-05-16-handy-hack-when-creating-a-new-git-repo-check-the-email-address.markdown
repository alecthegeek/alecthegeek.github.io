---
layout: post
title: "Handy Hack: Check the email address when creating a new Git repo"
date: 2013-05-16 13:11
comments: true
categories: git
---

__updated June 2013__ Added extra logic to use system Git if [Homebrew](http://mxcl.github.io/homebrew/) Git not installed
__updated July 2013__ Added extra logic to cd into repo dir before editing repo config

I was listening to [Git Minutes](http://episodes.gitminutes.com/2013/04/gitminutes-06-roberto-tyley-on.html)
recently when, as an aside, the perennial problem of setting the wrong email address
in a new repo was mentioned. This is a real nuisance as email addresses are attached to every commit.

A quick recap of how it works:

When you first install git you should set your global email address and user name by running the these two commands:

``` bash
git config --global user.name "My Name"
git config --global user.email my.name@email.domain
```

This adds two entries to the user's global git configuration files (`~/.gitconfig`). As a consequence, from now all
my commits will use this name and email address.

However, many people commit code to different repos using different email addresses, for instance a work address,
a personal address for side projects and
even a project specific email address for some cool FOSS project with its own domain.
This is not a problem as after creating the repo you can run the command

``` bash
git config user.email my.name@coolproject.domain
```

As the `--global` switch has been omitted this setting is local to the current repo (it's stored in `.git/config`)

This is all fine and dandy, unless you forget to set your email address correctly after creating a repo. So I wrote
a wrapper script to remind me.

It has a nice feature where you can pre-configure a list of email addresses that you pick from. If it's a new email address
then just type that instead. Type anything that does not look like an email address (e.g. 'x') to leave it unchanged.

To configure the list of valid email addresses run the following command

`git config --global user.emaillist "me@personal.address me@work.address me@coolFOSS.project`

Then place the following script early in your path

{% gist 5589064 %}
