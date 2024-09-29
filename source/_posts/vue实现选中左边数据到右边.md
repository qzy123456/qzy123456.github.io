---
title: vue实现选中左边数据到右边
date: 2024-09-29 17:34:16
tags: 前端
categories: vue
---
### vue
```
<template>
  <div class="container">
    <el-row>
      <el-col :span="4">
        <!-- 左边列表项 -->
        <div class="scrollable-menu">
          <el-menu
              class="el-menu-vertical-demo"
              background-color="#304156"
              text-color="#fff"
              active-text-color="#ffd04b"
          >
            <el-menu-item
                v-for="item in items"
                :key="item.id"
                :index="item.id.toString()"
                @click="selectItem(item)"
            >
              <i class="el-icon-menu"></i>
              {{ item.name }}
            </el-menu-item>
          </el-menu>
        </div>
      </el-col>
      <el-col :span="12">
        <!-- 右边选中的数据和数量 -->
        <div
            v-for="(selectedItem, index) in selectedItems"
            :key="index"
            class="item-container"
        >
          <div class="flex-container">
            <span class="item-name">{{ selectedItem.name }}</span>
            <el-input
                type="number"
                :min="1"
                v-model.number="selectedItem.quantity"
                class="quantity-input"
                @change="updateQuantity(selectedItem.id, selectedItem.quantity)"
            ></el-input>
            <el-button type="danger" @click="removeItem(index)">移除</el-button>
          </div>
        </div>
        <el-button type="primary" @click="addSelectedItem">提交</el-button>
      </el-col>
    </el-row>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        {id: 1, name: '数据1'},
        {id: 2, name: '数据2'},
        {id: 3, name: '数据3'},
        {id: 4, name: '数据4'},
        {id: 5, name: '数据5'},
        {id: 6, name: '数据6'},
        {id: 7, name: '数据7'},
        {id: 8, name: '数据8'},
        {id: 9, name: '数据9'},
        {id: 10, name: '数据10'},
        // 假设这里有很多数据项...
      ],
      selectedItems: [],
    };
  },
  methods: {
    selectItem(item) {
      const existingItem = this.selectedItems.find((x) => x.id === item.id);
      if (existingItem) {
        existingItem.quantity += 1;
      } else {
        this.selectedItems.push({...item, quantity: 1});
      }
    },
    removeItem(index) {
      this.selectedItems.splice(index, 1);
    },
    updateQuantity(id, quantity) {
      const item = this.selectedItems.find((x) => x.id === id);
      if (item) {
        item.quantity = quantity;
      }
    },
    addSelectedItem() {
      console.log(this.selectedItems)
    },
  },
};
</script>

<style scoped>
.container {
//margin: 30px;
}

.scrollable-menu {
  max-height: 400px; /* 可调整为适合的高度 */
  overflow-y: auto;
}

.item-container {
  margin-bottom: 10px;
}

.flex-container {
  display: flex;
  align-items: center;
  width: 100%;
}

.quantity-input {
  margin-right: 10px;
}

.item-name {
  margin-right: 10px;
}

.el-input {
  width: 50%;
}

.el-button {
  text-align: center;
}
</style>

```
### html版本
```angular2html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bootstrap 美化的数据列表示例</title>
<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
</head>
<body>

<div class="container">
  <div class="row">
    <div class="col-md-6" style=" height: 500px; overflow-y: scroll;">
      <!-- 左边列表项 -->
      <ul class="list-group" id="list-container">
        <li class="list-group-item" data-id="1">数据1</li>
        <li class="list-group-item" data-id="2">数据2</li>
        <li class="list-group-item" data-id="3">数据3</li>
      </ul>
    </div>
    <div class="col-md-6">
      <!-- 右边选中的数据和数量 -->
      <div class="input-group mb-3">
      </div>
      <ul class="list-group" id="selected-list">
        <!-- 右边选中的数据和数量将显示在这里 -->
      </ul>
    </div>
  </div>
</div>

<!-- 引入 Bootstrap 的 JavaScript 组件 -->
<script src="https://code.jquery.com/jquery-3.5.1.slim.min.js"></script> 
<script src="https://cdn.jsdelivr.net/npm/@popperjs/core@2.9.3/dist/umd/popper.min.js"></script> 
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script> 

<script>
document.addEventListener('DOMContentLoaded', function() {
            var listContainer = document.getElementById('list-container');
            var selectedList = document.getElementById('selected-list');
            // 监听左边列表项的点击事件
            listContainer.addEventListener('click', function(e) {
                var item = e.target.closest('.list-group-item');
                if (item) {
                    var itemId = item.getAttribute('data-id');
                    updateRightSide(itemId); // 更新右边的数据
                }
            });

            // 更新或增加右边的数据和数量
            function updateRightSide(itemId) {
                var itemText = '数据' + itemId;
                var selectedItem = selectedList.querySelector('li[data-id="' + itemId + '"]');
                var quantity = 1;

                if (selectedItem) {
                    // 如果右边已经有这个数据项，增加数量
                    quantity = parseInt(selectedItem.querySelector('input[type="number"]').value, 10) + 1;
                    selectedItem.querySelector('input[type="number"]').value = quantity;
                } else {
                    // 如果右边没有这个数据项，创建新的列表项
                    selectedItem = document.createElement('li');
                    selectedItem.className = 'list-group-item d-flex justify-content-between align-items-center';
                    selectedItem.setAttribute('data-id', itemId);
                    selectedItem.innerHTML = '<span>' + itemText + '</span>' +
                        '<input type="number" class="quantity form-control form-control-sm" value="' + quantity + '" min="1" style="max-width: 80px;">' +
                        '<button type="button" class="close" aria-label="Remove"><span aria-hidden="true">&times;</span></button>';
                    selectedItem.querySelector('.close').addEventListener('click', function() {
                        // 删除右边列表中的项
                        selectedList.removeChild(selectedItem);
                    });
                    selectedList.appendChild(selectedItem);
                }
            }
 
  // 监听添加按钮的点击事件
  addButton.addEventListener('click', function() {
    // 这里可以根据需要添加逻辑，比如更新数据库或执行其他操作
    // ...
  });
});
</script>

</body>
</html>
```
