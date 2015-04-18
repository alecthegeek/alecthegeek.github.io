---
layout: post
title: "alecthegeek's funky Go project layout tools"
date: 2015-03-16 10:02
comments: true
categories: golang
---

The most popular way to set up a Go development environment is have a single workspace (pointed to by `$GOPATH`) that contains all of the Go projects on a workstation (under a user's account). This means that a user can set their `$GOPATH` once, add `$GOPATH/bin` to their `$PATH` and your're good to go.

As each project is located under `$GOPATH/src/github.com/user/project` or similar.  In addition all of the various dependencies for every project are also inside the same tree. In order to make the separation between project code and external dependencies a lot clearer the `GOPATH` can contain two or more paths. `go get` then installs only into the 1st location on `GOPATH` list. Project code can then be stored in the 2nd PATH and `go build` etc. work fine.

This works well, but for me there are consequences I'd rather avoid.

1. I can't just move a project to a different location on my file system.
2. All of the dependencies for all my projects, and many Go tools, are located in my single project workspace.

So having a separate `GOPATH` for each project would be ideal, bit it's a PITA so have to set up a separate environment for each project

Automation to the rescue!

Here are my examples:

1. Install Go tools in a universal [location](https://gist.github.com/alecthegeek/3a48bb2bd3aa52a20306#file-go-install-tools) and have an easy way to install more.
2. Install the [direnv](http://direnv.net/) utility to manage up $GOPATH and $PATH correctly for each project.
3. Use a (rather large) bash script to setup new [projects](https://gist.github.com/alecthegeek/3a48bb2bd3aa52a20306#file-go-workspace), setup the direnv configuration and add new [packages]( https://gist.github.com/alecthegeek/3a48bb2bd3aa52a20306#file-go-package).


