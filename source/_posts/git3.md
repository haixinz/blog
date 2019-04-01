---
title: git 使用命令 3
tags: git
---


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