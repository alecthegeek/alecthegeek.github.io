---
layout: post
title: "Migrating from Wordpress.com to the Octopress blog engine"
date: 2013-05-16 18:11
comments: true
categories: blogging
---

I recently migrated this blog from [Wordpress.com](htpp://wordpress.com/) to the [Octopress](http://octopress.org/) blogging engine
with the result hosted [GitHub Pages](http://pages.github.com/).

The process s pretty well documented, but I thought I would pull it all together in one place.
I'm using a Mac OS X with [homebrew](http://mxcl.github.io/homebrew/) already installed.

1. Install Ruby as described by [Christopher Kaukis](http://www.interworks.com/blogs/ckaukis/2013/03/05/installing-ruby-200-rvm-and-homebrew-mac-os-x-108-mountain-lion)
2. Install Jykell as described [here](http://jekyllrb.com/docs/home/) and create a new blog. NB This will only be used for the migration.
3. Install the migration tool by following these [instructions](http://jekyllrb.com/docs/migrations/), but note that I had to install using the `--per` option.
4. Export your Wordpress blog to an XML file using the tools menu on Wordpress.com
5. Contime to follow the Jeykell migration instructions to import your Wordpress data into the temporary blog
6. Follow the [instructions](http://octopress.org/docs/setup/) to install Octopress and create an empty Octopress blog -- this will be your final blog.
7. Now from the temporary blog directory copy the cotents of the directores of `_posts`, `_attachements` and `_pages` to the corresponding
directoires which are located inside the `source` directory in your Octopress blog
8. Now deploy the blog as per the instructions in the Octopress documentation.
