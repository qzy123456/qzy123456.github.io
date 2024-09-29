---
title: Gitlab+GitlabRunner CI/CD
date: 2024-09-29 15:37:00
tags: git
categories: git
---
## 注意 gitlab跟runner版本要一致，不然会出问题
### docker安装gitlab
```
cd /opt/
mkdir gitlab
export GITLAB_HOME=/opt/gitlab
```
### 由于官方版本的gitlab/gitlab-ce:latest创建runner老是404，后来装了jh版本的。
```
docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 9001:80 --publish 2222:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  --shm-size 256m \
  registry.gitlab.cn/omnibus/gitlab-jh:latest
```
### 访问极狐 GitLab URL，http://192.168.252.131:9001,并使用用户名 root 和来自以下命令的密码登录：
```
[root@root~]# docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
Password: xxzIrC8HPFfuxVmGSyxxxx221Ihu+a2edEySMw=
```
### 登录，然后创建项目，上传公钥，省略。。。
### Docker方式安装注册gitlab-runner
```
docker run -d --name gitlab-runner --restart always \
  -v /opt/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```
### 创建runner,以及注册runner
![](1.png)
![](2.png)
![](3.png)
![](4.png)
### 进入runner容器内 docker exec -it gitlab-runner /bin/bash
```
gitlab-runner register  --url http://192.168.252.131:9001  --token glrt-Tz6onF8_bUSeNwaqqg8w
# 然后在交互界面，最后选择输入shell
```
## 极狐有个小坑
```
没有自带vim命令，要在容器内自行安装，其实也可在宿主机映射的目录修改
[root@249b6b18ffa8]# apt update
[root@249b6b18ffa8]# apt install -y vim
# 修改配置，增加clone_url配置，跟url并列
[root@249b6b18ffa8]# vi /opt/gitlab-runner/config/config.toml
[[runners]]
  name = "9b499a1ad4dc"
  url = "http://192.168.252.131:9001"
  id = 10
  token = "glrt-Tz6onF8_bUSeNwaqqg8w"
  token_obtained_at = 2024-03-07T03:43:37Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "shell"
  [runners.cache]
    MaxUploadedArchiveSize = 0
# 重启gitlab-runner所在容器
docker restart gitlab-runner
```
### 开始创建 .gitlab-ci.yml 官方示例，只是用来跑通项目
```
stages:          # List of stages for jobs, and their order of execution
  - build
  - test
  - deploy

build-job:       # This job runs in the build stage, which runs first.
  stage: build
  script:
    - echo "Compiling the code..."
    - echo "Compile complete."

unit-test-job:   # This job runs in the test stage.
  stage: test    # It only starts when the job in the build stage completes successfully.
  script:
    - echo "Running unit tests... This will take about 60 seconds."
    - echo "Code coverage is 90%"

lint-test-job:   # This job also runs in the test stage.
  stage: test    # It can run at the same time as unit-test-job (in parallel).
  script:
    - echo "Linting code... This will take about 10 seconds."
    - echo "No lint issues found."

deploy-job:      # This job runs in the deploy stage.
  stage: deploy  # It only runs when *both* jobs in the test stage complete successfully.
  environment: production
  script:
    - echo "Deploying application..."
    - echo "Application successfully deployed."

```
![](5.png)

