---
title: 解决git提交每次都要输入密码
date: 2024-09-29 15:35:26
tags: git
categories: git
---
### 如果您已将公钥添加到GitLab，并且仍然每次都要求输入用户名和密码，这可能是由于使用了HTTPS URL而非SSH URL来访问GitLab仓库。
### 当使用HTTPS URL时，GitLab将要求输入用户名和密码来进行身份验证，即使您已将SSH公钥添加到GitLab中。要解决这个问题，您需要将仓库的远程URL更改为SSH URL。
### 可以使用以下命令来更改Git仓库的远程URL：
```
git remote set-url origin git@your-gitlab-domain.com:yourusername/yourrepository.git
```
### 或者在我们的项目目录下打开控制台，输入
```
git config --global credential.helper store
```
### 然后生成一个.git-credentials，上边记录你的账号和密码，只需要输入一次用户名和密码，就会把账户信息保存到这个文件中。下次就不会弹出让你输入用户名和密码的提示啦
