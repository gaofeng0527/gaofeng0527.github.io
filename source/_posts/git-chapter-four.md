---
title: Git（分支管理）
date: 2018-11-23 16:05:54
tags: [Git]
---

# 分支管理

分支就像两条平行的线，互不干扰；在实际开发过程中，大家都是在分支上工作，互不干扰，而且每次修改后都可以在本地提交，而不用担心代码丢失。在需要时把分支合并，就可以完成协助开发了。

Git的分支是与众不同的，无论创建、切换和删除分支，Git在1秒钟之内就能完成！无论你的版本库是1个文件还是1万个文件。

# 分支工作原理

在Git仓库创建起始，会默认创建一个`master`分支。前面介绍版本回退的时候，引入了指针（`HEAD`）的概念，Git之所以与其他版本管理工具不同，一是因为Git是分布式管理工具，还一个是引入了指针的概念。其实我们在commit的时候，只是在移动指针，所以才会非常快。

严格来说`HEAD`不是指向提交的，而是指向`master`分支的，分支指向了提交，所以`HEAD`指向当前 分支如图：![git-br-initial](https://cdn.liaoxuefeng.com/cdn/files/attachments/0013849087937492135fbf4bbd24dfcbc18349a8a59d36d000/0)



每一次的修改提交，都被Git记录为一个节点，形成一个时间线，而每次提交当前分支就会指向最新的一次提交，而指针也指向当前分支。

<!-- more -->

当我们创建新`dev`分支的时候，`HEAD`会指向新的分支，然后新的分支在继续指向每次提交，如图：![git-br-dev-fd](https://cdn.liaoxuefeng.com/cdn/files/attachments/0013849088235627813efe7649b4f008900e5365bb72323000/0)

而`master`不变。

假如我们在`dev`上的工作完成了，就可以把`dev`合并到`master`上，就是直接把`master`指向`dev`的当前提交，就完成了合并：![git-br-ff-merge](https://cdn.liaoxuefeng.com/cdn/files/attachments/00138490883510324231a837e5d4aee844d3e4692ba50f5000/0)

Git完成分支合并也很快，只需要移动指针，而工作区的内容不需要变化。

# 操作命令

创建分支

>    $ git checkout -b dev
>    Switched to a new branch 'dev'  //创建了一个叫dev的分支，并切换到dev分支

>git branch dev
>
>git checkout dev   //效果和上面一样

查看分支

>    $ git branch
>
>    * dev
>      master
>

合并分支

需要先切换到主分支上

>    $ git checkout master
>    Switched to branch 'master'
>    Your branch is up-to-date with 'origin/master'.

>    $ git merge dev    //目标分支dev 合并到当前所在分支
>    Already up-to-date.

删除分支

>    $ git branch -d dev
>    Deleted branch dev (was d2a44dd).			

# 解决冲突

创建新分支 develop1.0 并在index.txt文件最末尾添加`create new branch develop`

>    $ git checkout -b develop1.0
>    Switched to a new branch 'develop1.0'
>
>    $ git add index.txt
>
>    $ git commit -m "develop branch mereg"
>    [develop1.0 961900a] develop branch mereg
>     1 file changed, 1 insertion(+)
>
>    ==============切换到master分支
>
>    $ git checkout master
>    Switched to branch 'master'
>    Your branch is up-to-date with 'origin/master'.
>
>    $ git branch
>      develop1.0
>
>    * master
>
>
>    ===============修改文件，并添加暂存区，提交到master分支版本库，查看状态，可以看到本地比远程仓库提前一个版本，可以使用git push命令提交到远程库
>
>    $ git status
>    On branch master
>    Your branch is ahead of 'origin/master' by 1 commit.
>      (use "git push" to publish your local commits)
>    nothing to commit, working directory clean
>
>    ======现在，同时在develop1.0 和master两个分支上修改了同一个文件，现在合并，看效果
>
>    $ git merge develop1.0
>    Auto-merging index.txt
>    CONFLICT (content): Merge conflict in index.txt
>    Automatic merge failed; fix conflicts and then commit the result.
>
>    //可以看到，合并失败，说有解决冲突后，重新提交。
>
>    $ git status
>    On branch master
>    Your branch is ahead of 'origin/master' by 1 commit.  //比远程库超前一个版本
>      (use "git push" to publish your local commits)
>    You have unmerged paths.  //合并失败，需要手动解决冲突
>      (fix conflicts and run "git commit")
>
>    Unmerged paths:
>      (use "git add <file>..." to mark resolution)
>
>    ​    both modified:   index.txt
>
>    no changes added to commit (use "git add" and/or "git commit -a")
>
>    ==============手动把冲突解决后，提交，查看状态
>
>    $ git status
>    On branch master
>    Your branch is ahead of 'origin/master' by 3 commits.
>      (use "git push" to publish your local commits)
>    nothing to commit, working directory clean
>
>    =========冲突已经解决，可以推送到远程仓库了
>
>    $ git push -u origin master
>    Counting objects: 7, done.
>    Delta compression using up to 4 threads.
>    Compressing objects: 100% (7/7), done.
>    Writing objects: 100% (7/7), 696 bytes | 0 bytes/s, done.
>    Total 7 (delta 3), reused 0 (delta 0)
>    remote: Resolving deltas: 100% (3/3), completed with 1 local object.
>    To git@github.com:gaofeng0527/learngit.git
>       704351c..010b68c  master -> master
>    Branch master set up to track remote branch master from origin.
>
>    =======再次查询状态，这个时候，工作区就干干净净了
>
>    $ git status
>    On branch master
>    Your branch is up-to-date with 'origin/master'.
>    nothing to commit, working directory clean

可以使用`git log --graph --pretty=oneline --abbrev-commit`命令查看合并记录

>    $ git log --graph --pretty=oneline --abbrev-commit
>
>    *   010b68c 解决冲突
>        |\
>        | * 961900a develop branch mereg
>    *   | c15f0c1 master branch test
>        |/
>    *   704351c conflict fixed
>        |\
>        | * 312319f develop1.0 commit
>    *   | 6d5613e master commit
>        |/
>    *   d2a44dd branch test
>    *   98b5e79 update readme.md
>
>        :...skipping...
>    *   010b68c 解决冲突
>        |\
>        | * 961900a develop branch mereg
>    *   | c15f0c1 master branch test
>        |/
>    *   704351c conflict fixed
>        |\
>        | * 312319f develop1.0 commit
>    *   | 6d5613e master commit
>        |/
>    *   d2a44dd branch test
>    *   4b05e65 暂时结束学习
>    *   37a791a one chapter commit
>