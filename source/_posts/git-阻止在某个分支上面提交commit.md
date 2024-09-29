---
title: git 阻止在某个分支上面提交commit
date: 2024-09-29 15:29:15
tags: git
categories: git
---
### 方法1 直接在git后台设置xxx分支为保护分支就可以了，比如master
### 方法2，进入到项目文件夹
```
cd .git/hooks/
cp pre-commit.sample pre-commit
```
### 在最前面加上代码,master可以替换对应分支
```
#!/bin/sh
branch="$(git rev-parse --abbrev-ref HEAD)"
if [ "$branch" = "master" ]; then
  echo "You can't commit to master!"
  exit 1
fi
# 后面的省略，可以不管，或者都删除了
```
