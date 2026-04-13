---
date: '2022-11-29T00:03:58+09:00'
draft: false
title: '【Git】HEAD^(キャレット)とHEAD~(チルダ)の違い'
tags: ["未分類"]
slug: '80'
---

[Git - gitrevisions Documentation](https://git-scm.com/docs/gitrevisions#_specifying_revisions) の以下の図がわかりやすい
```
G   H   I   J
 \ /     \ /
  D   E   F
   \  |  / \
    \ | /   |
     \|/    |
      B     C
       \   /
        \ /
         A

A =      = A^0
B = A^   = A^1     = A~1
C =      = A^2
D = A^^  = A^1^1   = A~2
E = B^2  = A^^2
F = B^3  = A^^3
G = A^^^ = A^1^1^1 = A~3
H = D^2  = B^^2    = A^^^2  = A~2^2
I = F^   = B^3^    = A^^3^
J = F^2  = B^3^2   = A^^3^2
```

[What's the difference between HEAD^ and HEAD~ in Git? - Stack Overflow](https://stackoverflow.com/a/2222920) の回答にある以下のスクリプトで上記のログを再現できる

```perl
#! /usr/bin/env perl

use strict;
use warnings;
use subs qw/ postorder /;
use File::Temp qw/ mkdtemp /;

my %sha1;
my %parents = (
  A => [ qw/ B C /               ],
  B => [ qw/     D E F /         ],
  C => [ qw/         F /         ],
  D => [ qw/           G H /     ],
  F => [ qw/               I J / ],
);

sub postorder {
  my($root,$hash) = @_;
  my @parents = @{ $parents{$root} || [] };
  postorder($_, $hash) for @parents;
  return if $sha1{$root};
  @parents = map "-p $sha1{$_}", @parents;
  chomp($sha1{$root} = `git commit-tree @parents -m "$root" $hash`);
  die "$0: git commit-tree failed" if $?;
  system("git tag -a -m '$sha1{$root}' '$root' '$sha1{$root}'") == 0 or die "$0: git tag failed";
}

$0 =~ s!^.*/!!;  # / fix Stack Overflow highlighting
my $repo = mkdtemp "repoXXXXXXXX";
chdir $repo or die "$0: chdir: $!";
system("git init") == 0               or die "$0: git init failed";
chomp(my $tree = `git write-tree`);      die "$0: git write-tree failed" if $?;

postorder 'A', $tree;
system "git update-ref HEAD   $sha1{A}"; die "$0: git update-ref failed" if $?;
system "git update-ref master $sha1{A}"; die "$0: git update-ref failed" if $?;
```

実際にやってみた

```sh
$ git show @~~ #=> D 
$ git show @~2 #=> D
$ git show @^^ #=> D
$ git show @^2 #=> C
```

- `~`も`^`もx世代前の1番目の親を指す
  - `~~` = `~2` = `^^` = 2世代前の1番目の親
- `^<n>`の場合だけx世代前のn番目の親を指す
  - `^2` = 1世代前の2番目の親

---

【参考】

- [Git - gitrevisions Documentation](https://git-scm.com/docs/gitrevisions#_specifying_revisions)
- [What is the difference between HEAD^ and HEAD~ in Git? - Stack Overflow](https://stackoverflow.com/questions/2221658/what-is-the-difference-between-head-and-head-in-git)
- [[git] チルダ(~)とキャレット(^)の違い | Tech控え帳](https://www.chihayafuru.jp/tech/index.php/archives/2535)
- [【やっとわかった！】gitのHEAD^とHEAD~の違い #Git - Qiita](https://qiita.com/chihiro/items/d551c14cb9764454e0b9)
