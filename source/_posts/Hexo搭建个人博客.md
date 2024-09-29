---
title: Hexo搭建个人博客
date: 2024-09-29 15:12:20
tags: Hexo
categories: Hexo
---

### 新建git项目
```
首先在 GitHub 新建一个仓库（Repository），名称为 https://username}.github.io，注意这个名比较特殊，
必须要是 github.io 为后缀结尾的。比如我的GitHub用户名就叫qzy123456，那我就新建一个qzy123456.github.io，新建完成之后就可以进行后续操作了。
```
### 安装 Hexo
```
npm install -g hexo-cli
```
### 初始化项目
```
hexo init {name}
```
### 安装git推送插件
```
npm install hexo-deployer-git --save
```
### 安装图片插件
```
npm install hexo-image-link --save
```
### 修改_config.yml中的post_asset_folder: true 这个修改可以同时在source/_posts目录下建立一个同名文件夹，用于放图片
### 配置站点信息_config.yml 文件，找到 Site 区域
```
title: 齐朝阳的博客
subtitle: 二级title
description: PHP、Go、Python、Js、Html、Vue、Mysql、Redis、Mongo、Linux、Es、ClickHouse、K8s、Docker、分布式...
keywords: "PHP、Go、Python、Js、Html、Vue、Mysql、Redis、Mongo、Linux、Es、ClickHouse、K8s、Docker、分布式..."
author: 技术栈
language: zh-CN
timezone: Asia/Shanghai
```
### 安装主题,在项目根目录
```
git clone https://github.com/theme-next/hexo-theme-next themes/next
```
### 修改项目根目录下的 _config.yml 文件，找到 theme 字段，修改为 next 即可，修改如下：
```
theme: next
```
### 主题配置，注意修改的 themes/next/_config.yml 文件
```
scheme: Pisces
```
### 新建页面
```
hexo new "Page Title"
```
### 生成静态文件
```
hexo g
```
### 启动本地服务
```
hexo s
```
### 删除静态文件缓存
```
hexo clean
```
### 发布到git,需要配置config
```
deploy:
  type: git
  rero: git@github.com:qzy123456/qzy123456.github.io.git
  branch: main
```
```
hexo deploy
```
### 新建 标签 和分类 页面
```
在根目录输入命令 hexo new page categories 会自动新建 categorier 文件夹并生成一个index.md文件，将里面的代码改为：
```
```
---
title: categories
date: 2020-01-21 12:12:26
type: "categories" 
---
```
### 同理，「标签」也一样 hexo new page tags 生成 tags 文件夹，其中会自动生成一个index.md文件，将代码改为：
```
---
title: tags
date: 2020-01-21 12:13:08
type: "tags"
---
```
### 配置菜单
```
在themes\next下的_config.yml中
menu:
  home: / || home
  #about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat

# Enable / Disable menu icons / item badges.
menu_settings:
  icons: true
```
### 使用时,如果要对文章使用「tags」「categories」，只需在文章开头添加如下代码：
```
---
title: 摸鱼
date: 2020-01-21 12:04:59
tags: 杂项
categories: 技术
---
```
### 为了方便，我们可以修改post模板，修改Hexo\scaffolds\post.md
```
---
title: {{ title }}
date: {{ date }}
tags:
categories:
---
```
### 其他
#### 用base编码处理图片
```
![Alt text](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA...)
```