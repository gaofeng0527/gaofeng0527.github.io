---
title: Git（一）
date: 2018-11-19 11:32:34
tags: [Git]
---

# Git

分布式版本管理系统，关键是免费。^_^

# windows install Git

直接从git官网下载安装程序即可，在开始-全部程序-git中有Git Bash 表示安装成功。

# Git config

查看本地user.name，user.email

>    $git config user.name
>
>    $git config user.email

设置本地global属性

>    $git config --global user.name "username"
>
>    $git config --global user.email "email@163.com"

# version manager

创建仓库，添加文件，提交版本

<!-- more -->

## mark repository

第一步需要创建一个空目录（也可以是在其他目录下，建议创建新的目录，个人习惯哈）

```shll
#mkdir learngit   ---创建目录
#git init      ---初始化成git仓库  pwd 命令可以查看当前目录
```

## add file

在该目录下创建一个文件readme.md

Git is a version control system.

Git is free software.

把文件添加到仓库中，然后在提交到仓库。

```shll
#git add readme.md
#git commit -m "fisrt git file"
```

*git commit -m "message"* 文件提交命令，这里为什么需要`add`和`commit`命令呢，因为`add`可以多次添加文件，而`commit`命令只需要执行一次，就可以把之前所有添加的文件提交到仓库中去。 `-m`参数表示本次提交的描述信息。

## change file

打开文件，修改第一行内容  Git is a distributed version control system.

这个时候，可以输入`git status`命令查看文件状态

>    $ git status
>    On branch master
>    Changes not staged for commit:
>      (use "git add <file>..." to update what will be committed)
>      (use "git checkout -- <file>..." to discard changes in working directory)
>
>            modified:   readme.md
>
>    no changes added to commit (use "git add" and/or "git commit -a")
>

会告诉你，内容被修改，并且还没有提交，可以使用`git add`命令添加修改，或者`git checkout`放弃修改。

也可以使用`git diff`命令查看具体修改

>    $ git diff
>    diff --git a/readme.md b/readme.md
>    index 02086ec..5dfce89 100644
>    --- a/readme.md
>    +++ b/readme.md
>    @@ -1,3 +1,3 @@
>    -git is a version control system,
>    +git is a distributed version control system,
>
>     git is a free software.
>    \ No newline at end of file
>    warning: LF will be replaced by CRLF in readme.md.
>    The file will have its original line endings in your working directory.
>

看到具体修改后，可以进行提交了，操作步骤和添加文件一样，先执行一次`git add`命令后，在查看一下状态

>    $git add readme.md

>    $ git status
>    warning: LF will be replaced by CRLF in readme.md.
>    The file will have its original line endings in your working directory.
>    On branch master
>    Changes to be committed:
>      (use "git reset HEAD <file>..." to unstage)
>
>            modified:   readme.md

>    $ git commit -m "change file after commit"
>    [master warning: LF will be replaced by CRLF in readme.md.
>    The file will have its original line endings in your working directory.
>    02df42d] change file after commit
>    warning: LF will be replaced by CRLF in readme.md.
>    The file will have its original line endings in your working directory.
>     1 file changed, 1 insertion(+), 1 deletion(-)

提交完成后，再次查看状态

>    $ git status
>    On branch master
>    nothing to commit, working directory clean

可以看到没有内容需要提交。由此总结一下：

1.   修改某个文件，查看状态，会提示有内容修改，并且是未提交状态
2.   add命令之后，查看状态，提醒文件已经添加，但是还没有提交
3.   commit 状态恢复成初始状态

# version manager

## 查看版本

多次提交以后，可以是使用`git log`查看往期版本

>    $ git log
>    commit 981e4f291d6d2174d47dbb78fe10cd4b40e8cf57
>    Author: gaofeng <gaofeng0527@163.com>
>    Date:   Mon Nov 19 14:19:27 2018 +0800
>
>        update file
>
>    commit 02df42d061245b3ee56514f0120079d5b2c9f798
>    Author: gaofeng <gaofeng0527@163.com>
>    Date:   Mon Nov 19 11:59:51 2018 +0800
>
>        change file after commit
>
>    commit f313c7c79377be500c5da315984a8a401ad9f582
>    Author: gaofeng <gaofeng0527@163.com>
>    Date:   Mon Nov 19 11:27:55 2018 +0800
>
>        wrote a readme file

`git log`命令显示从最近到最远的提交日志

如果觉得内容比较多，可以使用`git log --pretty=oneline`，把内容缩减在一行

>    $ git log --pretty=oneline
>    981e4f291d6d2174d47dbb78fe10cd4b40e8cf57 update file
>    02df42d061245b3ee56514f0120079d5b2c9f798 change file after commit
>    f313c7c79377be500c5da315984a8a401ad9f582 wrote a readme file

## 版本回退

首先我们需要知道，当前版本。在Git中使用*HEAD*表示当前版本，上一个版本*HEAD^*，上上个版本*HEAD^^*，之前n个版本的话，可以使用*HEAD~n*

按照之前的例子的话，最后一次提交应该是`update file`

回退到上一个版本:git reset --hard HEAD^

>    $ git reset --hard HEAD^
>    HEAD is now at 02df42d change file after commit

查看一下文件内容，是否是自己想要的版本。

版本回退成功之后，在用`git log`查看提交记录的话，就只有两个结果了

>    $ git log
>    commit 02df42d061245b3ee56514f0120079d5b2c9f798
>    Author: gaofeng <gaofeng0527@163.com>
>    Date:   Mon Nov 19 11:59:51 2018 +0800
>
>        change file after commit
>
>    commit f313c7c79377be500c5da315984a8a401ad9f582
>    Author: gaofeng <gaofeng0527@163.com>
>    Date:   Mon Nov 19 11:27:55 2018 +0800
>
>        wrote a readme file

我们回发现，在很早之前我们提交的那个版本找不到了，那如果想回到最后那个版本怎么办呢，有一个办法。如果之前的命令行窗口没有关的话，可以获取到想要去的那个commit id号。使用命令：`git reset --hard 981e4f`，981e4f是commit的前几位，不需要输入完整的commit id，git会自动去查找，但是也不能太简约，太简约了git如果找到多个版本，也不知道去哪里了。

>    $ git reset --hard 981e4f
>    HEAD is now at 981e4f2 update file

看版本又回到之前那个了。

如果命令行已经关了呢，这可咋办呢！！！

放心，Git提供了`reflog`命令，哇咔咔咔，这就可以找到之前的commit id了。

>    $ git reflog
>    981e4f2 HEAD@{0}: reset: moving to 981e4f
>    02df42d HEAD@{1}: reset: moving to HEAD^
>    981e4f2 HEAD@{2}: commit: update file
>    02df42d HEAD@{3}: commit: change file after commit
>    f313c7c HEAD@{4}: commit (initial): wrote a readme file

是不是可以舒口气了。