---
title: git 忽略idea文件夹
date: 2024-09-29 15:34:24
tags: git
categories: git
---
### 如果.gitignore文件不存在,在项目的根目录下创建一个名为.gitignore的文件，并在该文件中添加以下内容：
```
.idea/
```
### 如果.idea文件夹已经被跟踪，运行git rm --cached .idea来从Git跟踪中移除它，然后再提交这个更改。
```
git rm -r --cached .idea
git commit -m "xxxxx"
git push origin master
```

