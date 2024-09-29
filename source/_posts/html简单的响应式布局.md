---
title: html简单的响应式布局
date: 2024-09-29 17:39:47
tags: 前端
categories: html
---
### html版本
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>弹性盒容器-媒体查询-百分比布局--实现响应式布局</title>
    <style>
        body {
            font: 24px Helvetica;
            background-color: #fff;
        }
        /* 弹性盒容器 */

        .main {
            display: flex;
            min-height: 500px;
            margin: 0;
            padding: 0;
            flex-flow: row;
        }

        .main>* {
            margin: 4px;
            padding: 5px;
            border-radius: 7px;
        }

        article {
            background-color: #719dca;
            order: 2;
            flex-grow: 3;
        }

        nav {
            background-color: #ffba41;
            order: 1;
            flex-grow: 1;
            /* width: 20%; */
        }

        aside {
            background-color: aquamarine;
            order: 3;
            flex-grow: 1;
            /* width: 20%; */
        }

        header,
        footer {
            display: block;
            margin: 4px;
            padding: 5px;
            min-height: 100px;
            border: 2px solid #7e7234;
            border-radius: 7px;
            background-color: rgb(43, 144, 226);
        }

        footer {
            background-color: chocolate;
        }
        /* 媒体查询-当屏幕宽度小于XX，弹性盒容器内子元素按照纵轴方向排列 */

        @media screen and (max-width:640px) {
            .main {
                flex-flow: column;
            }
            article,
            nav,
            aside {
                order: 0;
            }
            nav,
            aside,
            header,
            footer {
                min-height: 50px;
                max-height: 50px;
            }
        }
    </style>
</head>

<body>
<header>弹性盒容器-媒体查询-百分比布局--实现响应式布局</header>
<!-- 弹性盒容器 -->
<div class="main">
    <!-- 弹性盒容器子元素 -->
    <article>文章</article>
    <nav>导航栏</nav>
    <aside>侧边栏</aside>
</div>
<footer>date:2023-12-19</footer>
</body>

</html>

```
### vue
```
<template>
  <el-container>
    <el-header>头部</el-header>
    <el-container>
      <!-- 准备两份aside侧边栏 -->
      <!-- 利用isCollapse动态控制侧边栏的宽度 -->
      <el-aside :width="isCollapse ? '64px' : '200px'" class="show">
        <!-- 切换侧边栏的显示与隐藏效果 -->
        <div class="toggle" @click="toggleCollapse">|||</div>
        <!-- collapse: 是否水平折叠收起菜单 -->
        <!-- collapse-transition：开启折叠动画 -->
        <el-menu :collapse="isCollapse" :collapse-transition="false" default-active="2" class="el-menu-vertical-demo" @open="handleOpen"
          @close="handleClose" background-color="#333744" text-color="#fff" active-text-color="#ffd04b">
          <el-submenu index="1">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>导航一</span>
            </template>
            <el-menu-item index="1-4">选项1</el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>主体</el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      // 控制侧边栏的显示与隐藏
      isCollapse: false // 默认显示侧边栏
    }
  },
  methods: {
    // 控制侧边栏的显示与隐藏
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    }
  }
}
</script>

<style lang="less">
html,
body,
.el-container {
  margin: 0;
  height: 100%;
}

.el-container {
  .el-header {
    padding: 0;
    background-color: #373d41;
    color: #fff;
  }

  .el-aside {
    background-color: #333744;

    // 默认只显示一份aside侧边栏
    &.show {
      display: block;
    }

    &.hide {
      display: none;
    }

    // 媒体查询，实现响应式
    @media (max-width: 992px) {
      // 取值与上面相反即可
      &.show {
        display: none;
      }

      // 取值与上面相反即可
      &.hide {
        display: block;
      }
    }

    .toggle {
      letter-spacing: 0.2em;
      color: #fff;
      text-align: center;
      line-height: 24px;
      background-color: #4a5064;
      cursor: pointer;
    }

    .el-menu {
      border-right: none;
    }
  }

  .el-main {
    background-color: #eaedf1;
  }
}
</style>

```
