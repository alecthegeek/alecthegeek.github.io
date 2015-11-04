---
layout: post
title: ! 'Handy Hack: Enter the office lottery syndicate with Perl'
tags:
- ego
- Perl
status: publish
type: post
published: true
---

If you are asked to chose numbers for a lottery here is a rather inelegant hack to create some entries.
Assume you need x numbers from a range of a to b inclusive then run this program as

`./lottery.pl  a b x`

Here is the script:

```perl
#!/usr/bin/perl
#

our %dup;

our $lower = $ARGV[0];
our $upper = $ARGV[1];
our $needed = $ARGV[2];

for (1..$needed) {
    my $x = int(rand($upper-1)+$lower);
    if ($dup{$x}) { redo }
    print  "$x \n";
    $dup{$x}++;
}
```
