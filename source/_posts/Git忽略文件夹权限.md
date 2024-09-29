---
title: Git忽略文件夹权限
date: 2024-09-29 15:29:15
tags: git
categories: git
---
### 项目范围
```
 git config core.fileMode false
```
### 也可以在git的全局范围内生效
```
git config --global core.filemode false
```
### 也可以通过修改 “~/.gitconfig” 文件，在其中添加下面内容

```
[core]
        filemode = false
```
