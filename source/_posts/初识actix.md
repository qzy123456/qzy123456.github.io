---
title: 初识actix
date: 2024-09-29 15:45:24
tags: rust
categories: rust
---
### 新建cargo项目
```
cargo new rust-web
```
### 编辑Cargo.toml
```
[dependencies]
actix-web = "4"
```
### 编写main.rs
```
use actix_web::{get,web, App, HttpServer, Responder,HttpResponse};

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
        .service(hello) //如果定义了get，post，直接用service
        .route("/index", web::get().to(indexs)) //如果没有定义，需要用route方法
    })
    .workers(8) // 设置工作线程数量
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

async fn indexs() -> impl Responder {
    HttpResponse::Ok().body("index")
}

#[get("/")]
async fn hello() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}
```
### 启动
```
cargo run
或者
cargo build
```
