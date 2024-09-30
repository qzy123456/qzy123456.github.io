---
title: Linux如何设置证书免密登录
date: 2024-09-29 17:44:47
tags: linux
categories: linux
---
### 执行ssh-keygen命令，生成id_rsa和id_rsa.pub两个文件，id_rsa是私钥（重要，需安全保管），id_rsa.pub是公钥，密钥生成过程中可根据提示对密钥设置密码，也可留空直接回车。
```
ssh-keygen -t rsa
```
### 创建authorized_keys文件并设置权限
```
[root@server1 ~]# touch ~/.ssh/authorized_keys
[root@server1 ~]# chmod 600 ~/.ssh/authorized_keys
```
### 将公钥内容追加到authorized_keys文件中
```
[root@server1 ~]# cd ~/.ssh
[root@server1 .ssh]# cat id_rsa.pub >> authorized_keys
```
### 修改 /etc/ssh/sshd_config文件，添加以下参数
### 注意 PermitRootLogin yes 这句会拦截root用户，可以先不写，等证书登录测试成功了加上
```
RSAAuthentication yes
PubkeyAuthentication yes
#PermitRootLogin yes  # 注意禁止root登录
AuthorizedKeysFile .ssh/authorized_keys
```
### 修改完配置文件，重启sshd服务
```
systemctl restart sshd
```
### 在Linux主机上登录验证
```
ssh root@localhost -i id_rsa
```
### 禁用密码登录,修改 /etc/ssh/sshd_config文件
```
PasswordAuthentication no
```
### 重启sshd服务
```
systemctl restart sshd
```

