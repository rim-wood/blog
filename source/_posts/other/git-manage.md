---
title: gitlab提交管理
date: 2017-08-09 19:30:02
toc: true
tags: 
- git
categories:
- other
---
在团队开发的项目中少不了gitlab作为代码管理库，遵循git的分支特性，总结了团队代码提交的一个标准，减少代码冲突带来的不可控因素。
<!--more-->

# gitlab提交管理

## 分支管理
每个大任务需要开启一个临时分支，从dev分支branch出来；branch的名称根据tapd中大任务的ID的命名

成员本地                                                       gitlab远程
                                                **临时**                            **稳定**             **发版**
                 |                               任务a分支<----->
local_branch_id  {      <----pull/push----->     任务b分支<-----MR(pass/refuse)---->   dev -----MR------> master            
                 |                               任务c分支<----->
## 创建分支

1. 拉取dev稳定版本的最新代码
```bash
    git pull origin dev
```

2. 创建本地分支并切换(tapd中大任务的ID)
```bash
    git checkout -b local_branch_id
```

3. 创建远程分支并关联(local_branch_id:remote_branch_id一致)
```bash
    git push origin local_branch_id:remote_branch_id
```

## 合并请求(Merge Request)
（gitlab不像github那样有类似于rebase merge操作，只能自己先rebase）
任务开发完毕之后，自己可能存在多次commit的情况，但是针对于这一个大任务，其实只需要一次commit。所以在合并之前，需要rebase之间的commit。
在合并之前，可能其他成员已经将代码合并到了dev，而自己的代码还是之前的，如果两人修改了同一个文件，直接提交合并请求，gitlab会提示存在冲突。
所以针对于上面两种情况：

1. 同步dev最新代码，解决冲突(pull and merge的语句就不写了)
2. 变基rebase操作

    - 首先使用git log 查看提交记录。
    - 然后选择从某一个提交记录开始变基
    
        **git rebase -i commitId（git log可看到的md5的一串字符）**

    - 输完之后，会让你选择如何合并，pick 是合并基础，也就是合并之后会提现在这一次commit上。 squash 是将本次提交合并到pick的提交上。主要是这两个命令，其他的不加以说明
    
    例如，下面我之前提交了三次；输入rebash之后进入VI界面，如果我需要合并到first上面，我就pick 第一个，后面的改为squash；然后esc，:wq保存

```bash
pick f8ff805 first
squash 7b0db2f second
squash 8dc017c third

# Rebase 9b9fd65..8dc017c onto 9b9fd65 (3 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit

```

保存之后，可以对这次rebase进行说明，同样是vi界面;添加说明之后也是:wq保存

```bash
# This is a combination of 3 commits.
大任务完成了，三个小任务包括
# This is the 1st commit message:

first

# This is the commit message #2:

second

# This is the commit message #3:

third

# Please enter the commit message for your changes. Lines starting
```

- 变基完成，假如之前有把那三次commit，push到远程对应的临时分支，直接push会失败，则使用
    
    **git push origin remote_branch_id -f** 

3. 提交到远程临时分支，并在gitlab上创建Merge Request

## 删除分支
code review 通过merge到了dev分支后，则可以删除掉远程分支

1. 用空分支的方式覆盖远程分支

    **git push origin :remote_branch_id**

2. 删除分支

    **git push origin --delete remote_branch_id**