---
layout: post
title: "Setting up MacVim for Go development using Mac Homebrew"
date: 2015-09-01 10:02
comments: true
categories: vim, homebrew
---

I still use [Vim](http://www.vim.org/), despite some other great Free editors out there including [Emacs](https://www.gnu.org/software/emacs/) and [Atom](https://atom.io/).

The reason for this is:

1. Works across all the platforms I use (including Raspberry Pi)
1. There is always a version of Vi installed on every machine I use (apart from Windows)
1. Works in a terminal session with tools like [tmux](https://tmux.github.io/)
1. Easy file navigation with command like `10j`, although it's only easy if you've developed the muscle memory
1. I'm used to it. See above
1. Has lots of plugins for programmers (Git, Go,...)

If you do use Vim then I recommend a plugin manager to make life simpler, I use [Vundle](https://github.com/VundleVim/Vundle.vim) installed from [Maximum Awesome](https://github.com/square/maximum-awesome)

I also use [Homebrew](http://brew.sh/) to manage my software packages, and if you are on OS X you should as well.  However it's taken a bit of effort to get Vim (installed via Homebrew) working the way I like it so hopefully these notes will help other people as well.

## MacVim Installation

The first thing is that you should install the version of MacVim that comes with Homebrew, _not_ [Cask](https://github.com/Homebrew/homebrew-cask). So assuming you already have Homebrew set up

`brew install macvim`

If MacVim gets upgraded you can have multiple versions of Vim installed
so let's make sure we have the latest version

`version=$(ls -1r /usr/local/Cellar/macvim/ | head -1)`

Now we need a link into `/Applications` so that we can use Vim as an OS X application bundle.

1. Delete any old version
  `[[ -d /Applications/MacVim.app ]] && sudo rm -rf /Applications/MacVim.app`

2. Add new link
  `ln -s /usr/local/Cellar/macvim/$version/MacVim.app /Applications/MacVim.app`

MacVim installs itself under various names. To see a list:

`ls /usr/local/Cellar/macvim/$version/bin`

On my machine I get the following list

```
lrwxr-xr-x  1 admin  admin     4 27 Jul 22:06 gview -> mvim
lrwxr-xr-x  1 admin  admin     4 27 Jul 22:06 gvim -> mvim
lrwxr-xr-x  1 admin  admin     4 27 Jul 22:06 gvimdiff -> mvim
lrwxr-xr-x  1 admin  admin     4 27 Jul 22:06 gvimex -> mvim
lrwxr-xr-x  1 admin  admin     4 27 Jul 22:06 mview -> mvim
-r-xr-xr-x  1 admin  staff  2286 27 Jul 22:06 mvim
lrwxr-xr-x  1 admin  admin     4 27 Jul 22:06 mvimdiff -> mvim
lrwxr-xr-x  1 admin  admin     4 27 Jul 22:06 mvimex -> mvim
```

So the actual executable file is `mvim`. Everything else just links to it.

> Side Note: The file `mvim` is a script that changes it's behaviour depending on the
> name it gets called.

The next thing I do is check that all of these links are also created in `/usr/local/bin`

```
for i in  $(ls /usr/local/Cellar/macvim/$version/bin)  ; do
    ls -l /usr/local/bin/${i##*/}
done
```

and I get the output

```
lrwxr-xr-x  1 admin  admin  33  1 Sep 06:50 /usr/local/bin/gview -> ../Cellar/macvim/7.4-77/bin/gview
lrwxr-xr-x  1 admin  admin  32  1 Sep 06:50 /usr/local/bin/gvim -> ../Cellar/macvim/7.4-77/bin/gvim
lrwxr-xr-x  1 admin  admin  36  1 Sep 06:50 /usr/local/bin/gvimdiff -> ../Cellar/macvim/7.4-77/bin/gvimdiff
lrwxr-xr-x  1 admin  admin  34  1 Sep 06:50 /usr/local/bin/gvimex -> ../Cellar/macvim/7.4-77/bin/gvimex
lrwxr-xr-x  1 admin  admin  33  1 Sep 06:50 /usr/local/bin/mview -> ../Cellar/macvim/7.4-77/bin/mview
lrwxr-xr-x  1 admin  admin  32  1 Sep 06:50 /usr/local/bin/mvim -> ../Cellar/macvim/7.4-77/bin/mvim
lrwxr-xr-x  1 admin  admin  36  1 Sep 06:50 /usr/local/bin/mvimdiff -> ../Cellar/macvim/7.4-77/bin/mvimdiff
lrwxr-xr-x  1 admin  admin  34  1 Sep 06:50 /usr/local/bin/mvimex -> ../Cellar/macvim/7.4-77/bin/mvimex
```

By the way Cask is still a fantastic add on for Homebrew an you definitely want use it. You can find my brewfile [here](https://github.com/alecthegeek/brewFile/blob/master/Brewfile).  You should modify to taste of course.

## Now let's set up some support for Go programming.

### Install Maximum Awsesome

```
git clone git@github.com:square/maximum-awesome.git
cd maximum-awesome/
rake
```

### Install You Complete Me

[YCM](https://valloric.github.io/YouCompleteMe/) provides word completion which is very handy.  However installing it a two stage process, assuming you are using Vundle installed using Maximum Awesome.

1. Add  `Plugin 'Valloric/YouCompleteMe'` to your `~/.vimrc.bundle.local` file

1. `cd maximum-awesome && rake`

1. Run `vim +PluginInstall +qall`

Now the tricky bit can be building YCM after it's installed, Hopefully it's as simple as `cd ~/.vim/bundles/YouCompleteMe && ./install.py --clang-completer`. If not follow the instructions on the website.

# Other Vim packages

I also install the following additional Vim packages

* fatih/vim-go
* SirVer/ultisnips
* epeli/slimux

I leave these as an exercise.

You now have a great general purpose editor with access to:

* Directory navigation (which some people refer to as project management) using NERDTree
* Powerful set of Git commands with Fugitive
* Lot's more goodies.

# Adding Go tools

The Vim-go package needs access to a few utilities which I prefer to install into a single location, as I have multiple GOPATHs.

So I need to run the Vim-go command `GoInstallBinaries` with GOBIN set to point to my Go install location, which is the value returned by `go env GOROOT`. N.B. Installing additional binaries in the official Go distribution may not be to your taste. But I simply run

`GOPATH=/tmp GOBIN=$(go env GOROOT)/bin vim -u ~/.vimrc  -c ":GoInstallBinaries" -c ":qall" /tmp/src/$$.go`

Now Go Hack and Profit!
