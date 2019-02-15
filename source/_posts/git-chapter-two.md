---
title: Git（二）
date: 2018-11-19 16:53:08
tags: [Git]
---

# 简介

前面一篇已经简单认识了Git，并且学习了8个基础的操作命令，今天继续学习

# 工作区和暂存区

Git和其他版本管理系统的区别就是，Git有暂存区的概念。

工作区：其实就是我们电脑上能看到的目录，比如之前创建的learngit目录。

版本库：工作区文件夹内有一个隐藏的.git文件夹，这个就是Git的版本库了。版本库中重要组成部分`stage`（或者叫`index`）的暂存区，还有创建的第一个分支`master`，以及指向`master`的指针`HEAD`（Git切换版本之所以快，是因为切换版本的时候，只是移动了该指针）。

<!-- more -->

![](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384907702917346729e9afbf4127b6dfbae9207af016000/0)

## 提交到暂存区

上一篇学习了`git add`和`git commit`命令是用来添加和提交文件的。

那么`git add`命令之后，发生了什么呢？我们来验证一下。

1.   在learngit目录中新建文件index.txt，并且修改readme.md文件

2.   查看工作区状态`git status`

     >    $ git status
     >    On branch master
     >    Changes not staged for commit:
     >      (use "git add <file>..." to update what will be committed)
     >      (use "git checkout -- <file>..." to discard changes in working directory)
     >
     >    ​    modified:   readme.md
     >
     >
     >    Untracked files:
     >      (use "git add <file>..." to include in what will be committed)
     >
     >    ​    index.txt
     >
     >    no changes added to commit (use "git add" and/or "git commit -a")

     可以看到`readme.md`文件被修改了，但是还没有提交

     index.txt文件为新建文件，状态为：Untracked

3.   使用`git add`添加文件

4.   再次使用`git status`查看一下状态

     >    $ git status
     >    warning: LF will be replaced by CRLF in readme.md.
     >    The file will have its original line endings in your working directory.
     >    On branch master
     >    Changes to be committed:
     >      (use "git reset HEAD <file>..." to unstage)
     >
     >      new file:   index.txt
     >      modified:   readme.md

     实际上，这个时候文件是提交到了暂存区中。如图：

     ![](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384907720458e56751df1c474485b697575073c40ae9000/0)

5.   `git commit`命令之后，添加的文件会一次性提交到初始创建的唯一分支`master`中，如图

     ![](https://cdn.liaoxuefeng.com/cdn/files/attachments/0013849077337835a877df2d26742b88dd7f56a6ace3ecf000/0)

     总结：`git add`命令实际上就是把要提交的所有修改放到暂存区（Stage），然后，执行`git commit`就可以一次性把暂存区的所有修改提交到分支。		

## 管理修改

Git之所以比其他管理工具优秀，是因为Git管理的是修改而非文件。`add` 和 `commit` 两个命令区分了工作区，暂存区和版本库三个区域。

例如：

我们修改文件index.txt，添加一行内容：*one line*，然后`add` 到暂存区

然后在修改index.txt ，在添加一行内容:*two line*

直接`git commit`，理论上这个时候使用`git status`查看状态，工作区应该是没有文件要提交的，下面来看一下

>    $ git status
>    On branch master
>    Changes not staged for commit:
>      (use "git add <file>..." to update what will be committed)
>      (use "git checkout -- <file>..." to discard changes in working directory)
>
>     modified:   index.txt
>
>    no changes added to commit (use "git add" and/or "git commit -a"

咦，怎么和预期的不一样呢？

因为之前我们已经了解过git有暂存区的概念，`add`命令只是把修改放到暂存区，而`commit`命令只把暂存区的内容提交到分支上。回想一下之前的操作，我们只在第一次修改的时候，执行的`git add`命令，因此，也只是把第一次的修改提交到了暂存区，那么`git commit` 命令只提交了第一次修改后放到暂存区的内容，因此第二次修改的内容，依然没有被提交。

总结：修改--》`git add`---》修改---`git add`---》。。。。--》`git commit`

使用`git diff HEAD -- index.txt`命令可以查看工作区和版本库里面最新版本的区别

>    $ git diff HEAD -- index.txt
>    diff --git a/index.txt b/index.txt
>    index 715646a..d2ff8dd 100644
>    --- a/index.txt
>    +++ b/index.txt
>    @@ -1 +1,2 @@
>    -one line
>    \ No newline at end of file
>    +one line
>    +two line
>    \ No newline at end of file

## 撤销修改

修改后的状态分为三种：1.修改后，并提交到缓存区  2.修改后并没有提交到缓存区  3.修改后已经commit了。

### 撤销工作区的修改

我们修改一下index.txt文件，添加一行 **20181123**，使用git status命令看一下状态

```shell
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   index.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

上面给出了提示，该文件被修改，但是还没有提交，可以使用git add `<file>`命令提交修改。也可以使用

git checkout -- `<file>`命令撤销修改。

```shell
$ git checkout -- index.txt
$ git status
On branch master
nothing to commit, working directory clean
```

可以看到修改被撤回了，文件也被修改了。

git checkout -- index.txt 命令 这里的`--`一定要加上，否则就成了切换分支了。

### 撤回暂存区的修改

如果修改已经被提交到暂存区了怎么办呢？，修改文件，敲命令看看情况

```shell
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   index.txt

no changes added to commit (use "git add" and/or "git commit -a")
========================
$ git add index.txt
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   index.txt

$ git checkout -- index.txt
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   index.txt
```

从上面的情况可以看出，git checkout -- index.txt 命令不能撤销暂存区的修改。因此要用命令`git reset HEAD <file> `可以把暂存区的修改撤销掉（unstage），重新放回工作区

```shell
$ git reset HEAD index.txt
Unstaged changes after reset:
M       index.txt
=============把修改重新放到了工作区，git status一下看看效果============================
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   index.txt

no changes added to commit (use "git add" and/or "git commit -a")
==========果然回来了，那么下一步是不是就知道怎么做了，对，使用checkout命令
$ git checkout -- index.txt
$ git status
On branch master
nothing to commit, working directory clean
```

是不是又修改回来了。

### 撤销版本库的修改

如果内容已经提交到版本库，那么上面的两种方式都不好用了，但是不要忘记我们之间学过的git reset 命令。

```shell
$ git reset --hard HEAD^
HEAD is now at 4b05e65 暂时结束学习
```

## 删除文件

```shell
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    a.txt

no changes added to commit (use "git add" and/or "git commit -a")
$ git rm a.txt
rm 'a.txt'
$ git commit -m "delete a.txt"
[master 4e5967d] delete a.txt
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 a.txt
$ git status
On branch master
nothing to commit, working directory clean

```

上面的命令可以看出，当工作区把文件物理删除后，使用git status查看状态，会有提示，有文件删除了。

这个时候，可以使用git rm a.txt命令把文件从版本库中删除，并使用git commit -m 命令提交修改。

如果删除错误的话，可以使用git checkout --a.txt 把文件拉回来，因为之前我们已经提交到版本库中。

