---
layout:       post
title:        "被遗忘的Git命令参数"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Git
---

> 撤销操作：
```java
git reset --soft xxx[hash 值]，--soft指撤销提交，但是保留对文件的修改。--hard值撤销提交，且不保留文件的修改。
```

> 提交之前，撤销某个文件的内容:
```java
git checkout -- <file>文件路径
```

> 将此次提交记录合并到上一次去：
```java
git commit --amend
```

> 自己在分支A, 想要将分支B中的内容合并进来：
```java
git merge B
```

> 删除分支
```java
git branch -d 分支名
```

> 设置别名
```java
配置的路径都在~/.gitconfig中
git config --global alias.s status 设置别名
git config --global unset alias.s 取消设置别名
也可以在终端设置别名：在全局~/.alias设置常用命令如下：

alias ga="git add"
alias gaa="git add ."
alias gc="git commit -m"
alias gs="git status"
```

> 暂存内容
```java
git stash 直接就会暂存
git stash apply stash@{0}可选 选用某个暂存内容 -> 【选用之后git stash list中还会存在，如果想要不存在就使用git stash pop xxx/git stash drop xxx】
git stash list 查看暂存列表
```

> 清除github Clone的项目操作如下：
```java
git config credential.helper "" 清除下载的认证信息
git config user.name xxx 设置名字
git config user.email xxx 设置邮箱
```

> 打印日志
```java
git log --oneline 【简介的形式展示】 --graph【图形化形式展示】
```

> Rebase的操作
```java
在分支B中，执行命令git rebase A, 意思就是将B中的操作记录，依次排列在A分支的结尾后面 【使得看上去就像在同一条分支上执行的操作】
回到分支A中，执行git merge B
```