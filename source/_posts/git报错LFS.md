---
title: git报错LFS
date: 2024-09-29 15:31:37
tags: git
categories: git
---
### 报LFS错
#### 错误1
```
WARNING: Authentication error: Authentication required: LFS only supported repository in paid enterprise.
```
#### 错误2
```
batch response: LFS only supported repository in paid enterprise.
```
### 第一个错误可以执行以下命令：命令中的{your_gitee}/{your_repo}是你的远程仓库地址，根据自己情况替换。
```
git config lfs.https://gitee.com/{your_gitee}/{your_repo}.git/info/lfs.locksverify false
```
### 第二个错误可以尝试删除./git/hooks/pre-push文件,最后重新push一下即可

