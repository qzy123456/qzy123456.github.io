---
title: vue-quill-editor富文本编辑器
date: 2024-09-29 17:32:52
tags: 前端
categories: 前端
---
### 安装
```
npm install vue-quill-editor --save
```
### 全局引入
```
import Vue from 'vue'
import VueQuillEditor from 'vue-quill-editor'
// 引入样式
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
Vue.use(VueQuillEditor)
```
### 指定vue文件中引入
```
// 引入样式
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
 
import { quillEditor } from 'vue-quill-editor'
 
export default {
  components: {
    quillEditor
  }
}
```
### 使用
```
<template>
  <quill-editor v-model="content"
                ref="myQuillEditor"
                :options="editorOption"
                @blur="onEditorBlur($event)"
                @focus="onEditorFocus($event)"
                @change="onEditorChange($event)">
  </quill-editor>
</template>
 
<script>
export default {
  data () {
    return {
      content: '',
      editorOption: {}, //编辑器配置项
    };
  },
  methods: {
    onEditorBlur () { }, // 失去焦点触发事件
    onEditorFocus () { }, // 获得焦点触发事件
    onEditorChange () { }, // 内容改变触发事件
  }
}
</script>
```

