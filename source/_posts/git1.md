---
title: git 使用命令 1
tags: git
---

- 创建本地版本仓库 

```bash
$ git init
```

- 把文件添加到版本库

```bash
$ git add readme.txt
```

- 把文件提交到版本仓库

```bash
$ git commit -m "wrote a readme file"
```
-m 参数后面输入的是本次提交的说明

为什么Git添加文件需要add，commit一共两步呢？因为commit可以一次提交很多文件，所以你可以多次add不同的文件，比如：

```bash
$ git add file1.txt
$ git add file2.txt file3.txt
$ git commit -m "add 3 files."
```

- 查看仓库的当前状态

```bash
$ git status
# On branch master
# Your branch is ahead of 'origin/master' by 2 commits.
#  (use "git push" to publish your local commits)
# Changes not staged for commit:
#  (use "git add <file>..." to update what will be committed)
#  (use "git checkout -- <file>..." to discard changes in working directory)

#        modified:   readme.md

# no changes added to commit (use "git add" and/or "git commit -a")

```
git status命令可以让我们时刻掌握仓库当前的状态，上面的命令告诉我们，readme.txt被修改过了，但还没有准备提交的修改。

- 查看readme.md 修改了哪些内容

```bash
$ git diff readme.md
```

- 查看git历史记录

```bash
$ git log (--pretty=oneline)  
```
参数为一行输出

- 退回上一个版本（退回上一百个版本）

```bash
$ git reset --hard HEAD^

$ git reset --hard HEAD~100

```

- 退回指定版本 (后跟版本号)

```bash
$ git reset --hard 3628164

```
现在，你回退到了某个版本，关掉了电脑，第二天早上就后悔了，想恢复到新版本怎么办？找不到新版本的commit id怎么办？

在Git中，总是有后悔药可以吃的。当你用$ git reset --hard HEAD^回退到add distributed版本时，再想恢复到append GPL，就必须找到append GPL的commit id。Git提供了一个命令git reflog用来记录你的每一次命令：

- 查看历史命令

```bash
$ git reflog

```

- 撤销修改

```bash
$ git checkout -- readme.txt

```
意思就是，把readme.txt文件在工作区的修改全部撤销


撤销修改分为以下三个场景：

场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD file，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。


- 删除文件

```bash
$ git rm test.txt

```
