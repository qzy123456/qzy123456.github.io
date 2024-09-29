---
title: git生成公钥
date: 2024-09-29 15:30:29
tags: git
categories: git
---
### 老是忘记  还是做个笔记把
### 命令，email@email.com替换成自己的邮箱
```
ssh-keygen -t rsa -C "email@email.com"
```
### 生成公钥位置
### windows
```
C:\Users\[用户名]\.ssh
```
### linux
```
~/.ssh
```
### 把id_rsa.pub的内容，复制到git的后台

