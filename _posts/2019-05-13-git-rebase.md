---
layout: post
title: "git rebase"
categories: [tools]
tags: [git, rebase]
redirect_from:
  - /2019/05/13/
---
* Kramdown table of contents
{:toc .toc}
# 修复以前的提交

起始状态，我们新提交了一个fixup greeting.txt 来修复以前的Add greeting.txt：

```
$ git log -2 --oneline
 (HEAD -> master) fixup greeting.txt
Add farewell.txt
Add greeting.txt
...
```

`git rebase -i HEAD~3` (`-i` for interactive). 弹出vim编辑器:

```
pick 8d3fc77 Add greeting.txt
pick 2a73a77 Add farewell.txt
pick 0b9d0bb fixup greeting.txt

# Rebase f5f19fb..0b9d0bb onto f5f19fb (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# f, fixup <commit> = like "squash", but discard this commit's log message
```

将补充提交放在被修改的Add greeting.txt后一个（此时显示顺序与git log相反），且修改补充提交为fixup或f：

```
pick 8d3fc77 Add greeting.txt
fixup 0b9d0bb fixup greeting.txt
pick 2a73a77 Add farewell.txt
```

保存后结果如下，补充提交消失，合入到Add greeting.txt：

```
$ git log -2 --oneline
fcff6ae (HEAD -> master) Add farewell.txt
a479e94 Add greeting.txt
```

# 将多次commit压缩成一个

用 `git rebase -i master` ，也可以用 `git rebase -i HEAD~12` 结果:

```
pick 1e85199 Add 'H' to squash.txt
pick fff6631 Add 'e' to squash.txt
pick b354c74 Add 'l' to squash.txt
pick 04aaf74 Add 'l' to squash.txt
pick 9b0f720 Add 'o' to squash.txt
pick 66b114d Add ',' to squash.txt
pick dc158cd Add ' ' to squash.txt
pick dfcf9d6 Add 'w' to squash.txt
pick 7a85f34 Add 'o' to squash.txt
pick c275c27 Add 'r' to squash.txt
pick a513fd1 Add 'l' to squash.txt
pick 6b608ae Add 'd' to squash.txt

# Rebase 1af1b46..6b608ae onto 1af1b46 (12 commands)
#
# Commands:
# p, pick <commit> = use commit
# s, squash <commit> = use commit, but meld into previous commit
```

将需要压缩的改为squash或s：

```
pick 1e85199 Add 'H' to squash.txt
squash fff6631 Add 'e' to squash.txt
squash b354c74 Add 'l' to squash.txt
squash 04aaf74 Add 'l' to squash.txt
squash 9b0f720 Add 'o' to squash.txtuq
squash 66b114d Add ',' to squash.txt
squash dc158cd Add ' ' to squash.txt
squash dfcf9d6 Add 'w' to squash.txt
squash 7a85f34 Add 'o' to squash.txt
squash c275c27 Add 'r' to squash.txt
squash a513fd1 Add 'l' to squash.txt
squash 6b608ae Add 'd' to squash.txt
```

保存后：

```
# This is a combination of 12 commits.
# This is the 1st commit message:

Add 'H' to squash.txt

# This is the commit message #2:

Add 'e' to squash.txt

# This is the commit message #3:

Add 'l' to squash.txt

# This is the commit message #4:

Add 'l' to squash.txt

# This is the commit message #5:

Add 'o' to squash.txt

# This is the commit message #6:

Add ',' to squash.txt

# This is the commit message #7:

Add ' ' to squash.txt

# This is the commit message #8:

Add 'w' to squash.txt

# This is the commit message #9:

Add 'o' to squash.txt

# This is the commit message #10:

Add 'r' to squash.txt

# This is the commit message #11:

Add 'l' to squash.txt

# This is the commit message #12:

Add 'd' to squash.txt

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Sun Apr 28 14:21:56 2019 -0400
#
# interactive rebase in progress; onto 1af1b46
# Last commands done (12 commands done):
#    squash a513fd1 Add 'l' to squash.txt
#    squash 6b608ae Add 'd' to squash.txt
# No commands remaining.
# You are currently rebasing branch 'squash' on '1af1b46'.
#
# Changes to be committed:
#	new file:   squash.txt
#
```

也可以用fixup调整：

```
Add squash.txt with contents "Hello, world"

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Sun Apr 28 14:21:56 2019 -0400
#
# interactive rebase in progress; onto 1af1b46
# Last commands done (12 commands done):
#    squash a513fd1 Add 'l' to squash.txt
#    squash 6b608ae Add 'd' to squash.txt
# No commands remaining.
# You are currently rebasing branch 'squash' on '1af1b46'.
#
# Changes to be committed:
#	new file:   squash.txt
#
```

结果：

```
commit c785f476c7dff76f21ce2cad7c51cf2af00a44b6 (HEAD -> squash)
Author: Drew DeVault 
Date:   Sun Apr 28 14:21:56 2019 -0400

    Add squash.txt with contents "Hello, world"
```

# 在git pull中使用rebase

通常在本地commit后，用`git pull`(`git fetch + git merge`)会生成一个merge的新提交。

```
git fetch origin
git merge origin/master
```

为避免这种情况发生，可以用

```
git fetch origin
git rebase origin/master
```

以上等效于`git pull --rebase`

如果足够喜欢这种方式，可以将rebase设为pull的默认选项，替代掉merge

```
git config --global pull.rebase true
```



**参考：<https://git-rebase.io/>**

