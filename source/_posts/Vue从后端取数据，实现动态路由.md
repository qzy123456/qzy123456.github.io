---
title: Vue从后端取数据，实现动态路由
date: 2024-09-29 17:41:38
tags: [Vue]
categories: [前端]
---
### 1.App.vue
#### 将获取菜单的方法放在全局中，以便每次刷新页面时，能够加载出。this.$store.state.userInfo是登入后存放在Vuex的用户信息
#### TODO：把数据放到本地存储，没有的时候再加载，这只是demo
```
<template>
  <div id="app">
    <router-view />
  </div>
</template>
 
<script>
import { generaMenu } from '@/assets/js/menu'
export default {
  created() {
    if (this.$store.state.userInfo != null) {
      generaMenu()
    }
  }
}
</script>
```
### 2.menu.js(@/assets/js路径下)
```
import Layout from '@/layout/index.vue'
import router from '@/router'
import store from '@/store'
import axios from 'axios'
import Vue from 'vue'
 
export function generaMenu() {
  axios.get('/api/admin/user/menus').then(({ data }) => {
    if (data.flag) {
      let userMenus = data.data;
      userMenus.forEach((item) => {
        if (item.icon != null) {
          item.icon = 'iconfont ' + item.icon
        }
        if (item.component == 'Layout') {
          item.component = Layout
        }
        if (item.children && item.children.length > 0) {
          item.children.forEach((route) => {
            route.icon = 'iconfont ' + route.icon;
            route.component = loadView(route.component)
          })
        }
      })
      store.commit('saveUserMenus', userMenus);
      userMenus.forEach((item) => {
        router.addRoute(item)
      })
    } else {
      Vue.prototype.$message.error(data.message);
      router.push({ path: '/login' })
    }
  })
}
 
export const loadView = (view) => {
  return (resolve) => require([`@/views${view}`], resolve)
}
```
### 3.SideBar.vue（侧边栏）
```
<template>
  <div>
    <el-menu
      class="side-nav-bar"
      router
      :collapse="this.$store.state.collapse"
      :default-active="this.$route.path"
      background-color="#304156"
      text-color="#BFCBD9"
      active-text-color="#409EFF">
      <template v-for="route of this.$store.state.userMenus">
        <template v-if="route.name && route.children && !route.hidden">
          <el-submenu :key="route.path" :index="route.path">
            <template slot="title">
              <i :class="route.icon" />
              <span>{{ route.name }}</span>
            </template>
            <template v-for="(item, index) of route.children">
              <el-menu-item v-if="!item.hidden" :key="index" :index="item.path">
                <i :class="item.icon" />
                <span slot="title">{{ item.name }}</span>
              </el-menu-item>
            </template>
          </el-submenu>
        </template>
        <template v-else-if="!route.hidden">
          <el-menu-item :index="route.path" :key="route.path">
            <i :class="route.children[0].icon" />
            <span slot="title">{{ route.children[0].name }}</span>
          </el-menu-item>
        </template>
      </template>
    </el-menu>
  </div>
</template>
 
<style scoped>
.side-nav-bar:not(.el-menu--collapse) {
  width: 210px;
}
.side-nav-bar {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  overflow-x: hidden;
  overflow-y: auto;
}
.side-nav-bar i {
  margin-right: 1rem;
}
*::-webkit-scrollbar {
  width: 0.5rem;
  height: 1px;
}
*::-webkit-scrollbar-thumb {
  border-radius: 0.5rem;
  background-color: rgba(144, 147, 153, 0.3);
}
</style>
```
