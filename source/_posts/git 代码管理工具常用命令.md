---
title: git 代码管理工具常用命令
categories: Tools 
tags: git 
date: 2019-09-18 12:00:00
update: 2020-11-16 12:00:00
cover: https://i.imgur.com/nUYzQ6Y.jpg
---


首先区分git是分布式代码管理系统，每个计算机上或者说每个克隆的git仓库都是完整的，因此同一个本地克隆的远程的git仓库的关系是，本地对应远程（远端，非本机的），因此git的命令行也就对应了本地仓库的操作和同步到远程仓库的操作。

## git基础配置

假设在git服务器上设置的用户名是：gituser1，邮箱是：gituser1@git.com

### 配置git提交时候的全局用户名

```git
gitconfig --global user.name gituser1
```

### 配置git提交时候的全局邮箱

```git
git config --global user.email gituser1@git.com
```

### 配置gerrit的review

```git
git config --global review.gerrit.example.com:8081.username yourname
```



## 常用仓库操作

### 创建git库

```git
git init
```

### 查看本地仓库分支

```git
git branch
```

想要查看对应的远端仓库分支可以用`git branch -a`来查看

### 拉去仓库并创建分支

```git
git checkout -b master master_copy
```

mater对应是远程仓库的分支名，master_copy是我们需要创建的本地仓库的分支名

### 查看当前git库状态

```git
git status
```

显示出红色的文件代表当前仓库中有文件与上一次同步状态不同
如果没有文件显示，且没有更改，会显示当前分支与远端分支一致，没有需要提交的更改。
如果显示没有更改，但是当前分支与远端分支不同步，则需要同步远端分支

*养成良好的习惯提交修改代码前先把本地仓库与远端仓库同步*

```git
git remote -v
```
查看当前git仓库对应的远程git仓库

### 同步本地仓库与远端仓库

```git
git pull
```

这个时候有可能报错，说明本地仓库之前就已经发生混乱，一般是本地有修改，又去同步，产生文件修改冲突后没有解决，如果确认可以把本地修改重置，则可以使用

```git
git pull --rebase
```

 这个命令会把本地仓库换一个基础，可以认为是仓库是需要一个地皮的，rebase的作用就是把仓库换个干净的地皮重新造一个一样的与远程一样的仓库，　这样新的仓库就与远端的仓库完全一致了。

但是还有情况会出现本地仓库领先与远端仓库，而我们又不需要本地的修改，就可以把远端仓库检查校对拷贝过来。比如文件名为temp.log（相当于是把本地改动回退到远端文件版本）

```git
git checkout　temp.log
```
将当前文件夹内所有文件都回退到远端git仓库
```git 
git checkout .
```

---

当修改了名为temp.log的文件后

### 对比本地修改文件与本地仓库之间的区别

```git
git diff temp.log
```

 终端会显示出该文件的修改点，对应增删，如果开启了git的高亮显示，会有绿色表示添加，红色表示删除

 可以使用`git config --global color.ui true`来开启git颜色高亮

### 添加文件(该文件在当前目录下)到当前git仓库（确切说是添加到了本地仓库的提交暂存区）

```git
git add temp.log
```

将当前文件夹内所有文件都添加到当前git仓库

```git
git add .
```

 将文件添加到当前git仓库后，再使用`git status`查看状态，会发现文件对应路径会从红色变成绿色。

### 将添加到本地仓库的文件提交到本地仓库（从暂存区真正提交到仓库，会从暂存区移除改文件）

```git
git commit temp.log
```

&

```git
git commit .
```

&

```git
git commit temp.log -m "comit message(此引号内应填入提交信息)"
```

 提交后可以可以通过`git log`命令查看这个git仓库的历史记录

### 查看当前本地仓库的历史提交记录

```git
git log
```

### 将本地仓库回退到历史记录中史上一次的修改状态

```git
git reset HEAD^
```

 解释下这个命令，reset本身可以重置，而HEAD表示分支的头，^表示分支头部的上一个位置，所以是回退到历史记录的上一条，也就是说如果你`git pull`后本地没有修改，但是这个时候执行了本条命令，那么你当前的本地仓库还是会回到历史记录中的上一个位置（尽管这个提交不是你操作的）

### 修改本地仓库的历史记录中上一次的提交信息（实际是合并上一次的提交）

```git
git commit --amend
```

可以使用`git reset HEAD^`命令使当前仓库回退到历史记录中上次的提交状态，这个时候比如说我们的temp.log已经commit后，使用这个resset命令会让我们的commit失效，文件状态会变成add之前的状态，这个时候我们可以修改temp.log然后使用`git commit --amend`合并上一次提交。



## 其他git命令（进阶）

### 显示上一次提交的日志

```git
git show HEAD^
```

### 显示某次提交的日志

```git
git show dfb02e6e4f2f7b573337763e5c0013802e392818
```

### 对比上一次提交的不同

```git
git diff HEAD^
```

### 显示所有已添加index（暂存）但还未commit的变更

```git
git diff --cached
```

### 切换到dev分支（当前不在dev分支）

```git
git checkout dev
```

### 获取所有远程分支

```git
git fetch
```

### 删除本地分支

```git
git branch -d dev
```

强制删除 `git branch -D dev`

### 图示当前分支历史

````git
git show-branch
````

图示所有分支`git show-branch -all`

### 单独提交到有gerrit系统的代码审核

```git
git push [remote_alias] HEAD:refs/for/[branch_name]
```

替换`[remote_alias]`和`[branch_name]`

 remote_alias查看方法： `git remote -v`

 **千万不要使用`git push  [branch_name]:origin/master`**

### 查看git仓库中文件某些行代码的最新提交信息

```git
git blame -L 25,+5 xxxx.java
```

   -L 参数表示以行数参数查看，25表示想要查看的行数，+5表示从25行开始看到30行的代码