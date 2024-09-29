---
title: Vue3+Elementplus 递归菜单展示
date: 2024-09-29 17:35:21
tags: 前端
categories: vue
---
### 这里只是做个笔记，js，css那些都没写
### 子组件 MenuItem
```
<template>
  <template v-if="item.children">
    <el-sub-menu :index="item.value">
      <template #title>{{ item.label }}</template>
      <MenuItem v-for="childItem in item.children" :key="childItem.value" :item="childItem" />
    </el-sub-menu>
  </template>
  <template v-else>
    <el-menu-item :index="item.value">{{ item.label }}</el-menu-item>
  </template>
</template>
<script setup>
import { defineProps } from 'vue';
 defineProps(['item']);
</script>
<style scoped>
</style>

```
### 父组件,调用
```
<template>
  <el-menu
      :default-active="activeIndex"
      class="el-menu-vertical-demo"
      @select="handleSelect"
      background-color="#f8f8f9"
      style="margin-top: 20px;margin-left: 1px;width: 20%"
  >
    <MenuItem v-for="menuItem in menuItems" :key="menuItem.value" :item="menuItem" />
  </el-menu>
</template>
<script setup>
import MenuItem from "./components/MenuItem.vue";
import {ref} from "vue";
const menuItems = [
  {
    value: '1',
    label: '菜单1',
    children: [
      {
        value: '1-1',
        label: '子菜单1-1',
        children: [
          { value: '1-1-1', label: '子菜单1-1-1' },
          { value: '1-1-2', label: '子菜单1-1-2' },
        ],
      },
      { value: '1-2', label: '子菜单1-2' },
    ],
  },
  {
    value: '2',
    label: '菜单2',
    children: [
      { value: '2-1', label: '子菜单2-1' },
      {
        value: '2-2',
        label: '子菜单2-2',
        children: [
          { value: '2-2-1', label: '子菜单2-2-1' },
          { value: '2-2-2', label: '子菜单2-2-2' },
        ],
      },
    ],
  },
  {
    value: '3',
    label: '菜单3',
    children: [
      {
        value: '3-1',
        label: '子菜单3-1',
        children: [
          {
            value: '3-1-1',
            label: '子菜单3-1-1',
            children: [
              { value: '3-1-1-1', label: '子菜单3-1-1-1' },
              { value: '3-1-1-2', label: '子菜单3-1-1-2' },
            ],
          },
        ],
      },
    ],
  },
];
const activeIndex = ref(1)
const handleSelect = async (key, keyPath) => {
  console.log(key, keyPath)
  this.activeIndex = key
}
</script>
```

