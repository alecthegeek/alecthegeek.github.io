---
layout: post
title: ! 'Handy Hack: Calculating Git SHA1 on files without Git installed'
tags:
- Change Audit
- Git
- LinkedIn
- Linux
- Mac
- Perl
- Software Configuration Management
- Software Development
status: publish
type: post
published: true
meta:
  _edit_last: '451289'
  email_notification: '1268788540'
---

If you are not a Git geek this will be of limited interest and I urge you "move along"

Updated after tips from <a title="uselss use of a cat" href="http://partmaps.org/era/unix/award.html#cat" target="_blank">Randal Schwartz</a> and <a title="Tip on Stackoverflow" href="http://stackoverflow.com/questions/552659/assigning-git-sha1s-without-git/1213226#1213226" target="_blank">Charles Bailey</a>


Still here? OK. As you will know every file stored in a Git repo is identified by a <a href="http://en.wikipedia.org/wiki/SHA_hash_functions">SHA1</a> hexadecimal string (as <a href="http://progit.org/book/ch9-2.html">explained</a> in the <a href="http://progit.org/">Pro Git</a> book, it's the hash of the concatenation of the  file contents and a Git specific header). You can calculate the Git SHA1 of any file with the command

```bash
git hash-object $file
```

assuming you have Git installed

If you do not have access to Git, but do have access to a UNIX style shell or Perl you have a couple of options (the Pro Git book also shows a Ruby solution and <a title="Sha1 calculation on Stackoverflow" href="http://stackoverflow.com/questions/552659/assigning-git-sha1s-without-git" target="_blank">Stackoverflow</a> has examples in Python and #F))


On Cygwin, OS X or Linux

```bash
(printf "blob %s0" $(wc -c < $file);cat $file)|sha1sum -b | cut -d " " -f 1
```

(N.B. Assumes GNU tools installed on OS X)

On Solaris

```bash
(printf  "blob %s0" $(wc -c < $file);cat $file)|digest -a sha1 | cut -d " " -f 1
```


In Perl

```perl
#!/usr/bin/env perl

# See also Git::PurePerl at http://search.cpan.org/dist/Git-PurePerl/

use strict;
use warnings;
use Digest::SHA1;

my @input = <>;

my $content = join("", @input);

my $git_blob = "blob " . length($content) . "\0" . $content;

my $sha1 = Digest::SHA1->new();

$sha1->add($git_blob);

print $sha1->hexdigest();
```
