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


- 设置远程仓库

第1步：创建SSH Key。在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有id_rsa和id_rsa.pub这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：

```bash
$ ssh-keygen -t rsa -C "youremail@example.com"
```
如果一切顺利的话，可以在用户主目录里找到.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。

第2步：登陆GitHub，打开“Account settings”，“SSH Keys”页面，然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容

- 添加远程库

现在的情景是你已经在本地创建了一个Git仓库后，又想在GitHub创建一个Git仓库，并且让这两个仓库进行远程同步，这样，GitHub上的仓库既可以作为备份，又可以让其他人通过该仓库来协作。

首先登陆GitHub，然后，在右上角找到“Create a new repo”按钮；

在Repository name填入仓库名称（和本地文件夹名称，如learngit），其他保持默认设置，点击“Create repository”按钮，就成功地创建了一个新的Git仓库；

然后 在本地运行如下命令，用以把本地库和远程库关联

```bash
$ git remote add origin git@github.com:haixinz/learngit.git

```
最后，把本地的文件推送到GitHub

```bash
$ git push -u origin master
```

从现在起，只要本地作了提交，就可以通过命令：

```bash
$ git push origin master
```
把本地master分支的最新修改推送至GitHub，现在，你就拥有了真正的分布式版本库！

- 从远程库删除

```bash
$ git add -A
```

```bash
$ git commit -m 'del'
```

```bash
$ git push origin master
```
git add . ：他会监控工作区的状态树，使用它会把工作时的所有变化提交到暂存区，包括文件内容修改(modified)以及新文件(new)，但不包括被删除的文件。

git add -u ：他仅监控已经被add的文件（即tracked file），他会将被修改的文件提交到暂存区。add -u 不会提交新文件（untracked file）。（git add --update的缩写）

git add -A ：是上面两个功能的合集（git add --all的缩写）


- 从远程仓库克隆

```bash
$ git clone git@github.com:haixinz/learngit2.git
```


- 创建分支

首先，我们创建dev分支，然后切换到dev分支：

```bash
$ git checkout -b dev

Switched to a new branch 'dev'
```

git checkout命令加上-b参数表示创建并切换，相当于以下两条命令：

```bash
$ git branch dev
$ git checkout dev
Switched to branch 'dev'
```

- 查看当前分支

```bash

$ git branch
* dev
  master
```

- 在当前分支提交

```bash
$ git add readme.txt 
$ git commit -m "branch test"
```

- 把dev分支合并会master分支

先切换回master分支

```bash
$ git checkout master

```
把dev分支的工作成果合并到master分支上

```bash
$ git merge dev

```

- 删除dev分支

```bash
$ git branch -d dev

```

因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在master分支上工作效果是一样的，但过程更安全。
