---
title: Rust编写wasm入门
date: 2024-09-29 15:42:39
tags: rust
categories: rust
---
### 创建项目
```
cargo new --lib my-wasm
```
### 添加依赖Cargo.toml
```
[dependencies]
wasm-bindgen = "0.2"

[lib]
crate-type = ["cdylib"]
```
### 编写代码 src/lib.rs
```
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn add(a: i32, b: i32) -> i32 {
       a + b
}
```
### 安装扩展，构建wasm
```
## 添加 wasm-pack 
cargo install wasm-pack
## 构建
wasm-pack build --target web
```
### pkg文件夹
![](1.png)
### 编写html测试页面
```
<!DOCTYPE html>
<html lang="en">
    <head>
    <meta charset="UTF-8">
    <title>Rust Wasm Example</title>
    <script type="module">
        import init, { add } from './pkg/my_wasm.js';
            async function run() {
                await init();
                console.log(add(2, 3));
            }
           run();

    </script>
    </head>
    <body>
        <h1>Rust Wasm Example</h1>
    </body>
</html>
```
![](2.png)
### html需要在服务器环境下打开，如果以文件方式打开，会报错跨域,我这里直接用go做文件服务器了
```
package main

import (
	"fmt"
	"net/http"
)

func main() {
	// 设置文件服务器的根目录
	http.Handle("/", http.FileServer(http.Dir(".")))
	// 定义服务器监听的端口
	port := "8080"
	// 启动服务器
	fmt.Printf("Starting file server on http://localhost:%s/\n", port)
	err := http.ListenAndServe(":"+port, nil)
	if err != nil {
		fmt.Println("Error starting file server: ", err)
	}
}
```
