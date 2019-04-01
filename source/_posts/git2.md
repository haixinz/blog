---
title: git 使用命令 2
tags: git
---


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

