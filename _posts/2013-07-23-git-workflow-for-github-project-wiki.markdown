---
layout: post
title: "Git workflow for GitHub Project Wiki"
date: 2013-07-23 20:02
comments: true
categories: GitHub
---

Update 2016-08-10:  I've been using this workflow a lot over the last 18 months and decided it's easier to have two seperate repos (one for code and one for wiki documentation). This allows you to work on both sets of content in parallel rather than keep checking each branch out.

So please read the following suggestions with that caveat.

When you create a project repo in GitHub you can get get a handy project wiki. This is a great place
to add some documentation. However it's not part of your main project artifacts. You can either edit
the wiki directly via the web interface or use these instructions to manage the content via Git. However
the Git option it's a bit clunky as the wiki source is managed via a separate repo (e.g. myProject.wiki.git).

Inspired by [@AdamTuttle](https://twitter.com/AdamTuttle)'s [post](http://fusiongrokker.com/post/how-you-can-contribute-to-taffy-documentation)
I came up with the this workflow which has the following features:

* Single repo for code _and_ project wiki.
* Version control of wiki changes and integration management
* Uses the GitHub pull request system.

Downside is setup and keeping track of branches. Here are the draft step
by step instructions.

I'm using a project called ``kewlProject`` as an example, and it's written from the
perspective of a project contributor (called ``me``). An integration manager (called ``intermgr``)
runs the main repo
that houses the published wiki. _Special note_: This workflow assumes that
integration manager is not using the same workflow and is handling the wiki using a dedicated
repo and the master branch.


1. On GitHub fork the project
  ``https://github.com/intermgr/kewlProject``. Now we have
  ``https://github.com/me/kewlProject``.

    Now on your workstation


2. ``git clone git@github.com:me/kewlProject.git``

3. ``git co --orphan wiki``

4. remove all files except ``.git/`` with ``git rm -fr .``


8. ``git remote add test_wiki
git@github.com:me/kewlProject.wiki.git``

7. ``git remote add upstream_wiki
git@github.com:intermgr/kewlProject.wiki.git``

9. ``git pull upstream_wiki master:wiki``
N.B. If the integration manager is using the same scheme as you then you should pull from their ``wiki`` branch as instead

10. ``git push test_wiki wiki:master``

      Now my repo wiki will be populated on GitHub website

      To make a change

11. ``hack hack``

11. ``git add . && git commit``

12. ``git push test_wiki wiki:master``

13. Review change on GitHub web

14. ``git pull upstream_wiki master:wiki && git push origin wiki``

15. Create pull request on GitHub from your wiki branch to integration master

Using this system I should be able to ``git checkout master`` and hack code as
well.
