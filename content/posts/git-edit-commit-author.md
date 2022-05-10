---
title: "git修改提交用户信息"
date: 2022-05-10T14:08:34+08:00
tags: ["git"]
draft: false
---

- `git`更新历史提交用户信息

<!--more-->

### 问题

电脑上有几个不同的`git`角色，经常新仓库忘记设置`git`信息，把错误的用户信息提交上去。

### 修改当前仓库用户信息

```shell
git config --local user.name <name>
git config --local user.email <email>
```

### 修改最近一次commit信息

1. 方法1。`git commit --amend` 进去修改，然后`vim :wq`保存退出
2. 方法2。`git commit --amend --author="name <email>" --no-edit`

### 修改历史提交信息

1. `git log`查看想要修改的`commit`。记录下想要修改的`N`个`commit`

2. `git rebase -i HEAD~n`  n为往前`rebase`n个`commit`，n > 要修改的最前一个`commit`的位数。

   <img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220510142024.png" width="600"/>

   

   - 根据第一步的记录，把需要修改的`commit`，`pick`改为`edit`。`:wq`保存退出。有如下输出

     ​	<img src="https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/20220510142210.png" width="800"/>

3. 此时提示`停止在 aae578c... feat: redis design and implement`。表示你可以对该commit进行修改。

   1. 修改`commit`信息

      - 方式1。`git commit --amand` 进入交互编辑修改`commit`信息

      - 方式2。`git commit --amend --author="name <email>" --no-edit`直接修改用户信息

   2. 保存该`commit`的修改

      - `git rebase --continue`

   3. 重复以上`1.2.`操作，直到所有`edit`的`commit`都修改完成。

4. 强制更新远程仓库。

   - `git push --force`
