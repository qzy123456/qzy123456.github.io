---
title: express + mysql + jwt 登录+简单的增删改查
date: 2024-09-29 17:38:12
tags: [前端]
categories: [nodejs,mysql,jwt]
---
### gitee地址 https://gitee.com/newly-released_0/express-mysql-jwt
### jwt的代码
```
const express = require('express')
const app = express()
const compression = require('compression');
const cors = require('cors')
// 导入 jsonwebtoken 和express-jwt 两个包
const jwt = require('jsonwebtoken')
const {expressjwt:jwt2}= require('express-jwt')

var usersRouter = require("./controllers/UserController");
// 创建secret对象
const secretKey = 'tokenKey123 num.1 ^_^'

require('./utils/db.js')

app.use(express.urlencoded({ extended: false }));
app.use(express.json())

app.use(cors())
// 压缩
app.use(compression());

// 注册将JWT字符串解析还原成JSON对象的中间件
// .unless()指定不需要访问权限的接口
app.use(jwt2({ secret: secretKey,algorithms:["HS256"] }).unless({ path: [/^\/user\//,'/api/login'] }))

app.use('/user', usersRouter);
// 编写登陆接口
app.post('/api/login',(req,res)=>{
    const userinfo = req.body

    // 登陆成功 => 将用户信息转换成token，并设置有效期
    const token = jwt.sign({username:userinfo.username},secretKey,{expiresIn:'300s'})
    res.send({
        staus:200,
        msg:'登陆成功！',
        token:token
    })
})

app.get('/admin/getinfo',(req,res)=>{
    console.log(req.auth);
    res.send({
        staus:200,
        message:'获取用户信息成功！',
        datas :req.auth.username
    })
})

// 全局处理404
app.use('*', (req,res) => {
    res.send({
        status : 200,
        msg : "404 , not found",
        data : ''
    })
})

//最后使用错误中间件
app.use((err, req, res, next) => {
    console.error(err.stack);
     // 这次失败是由token解析失败导致的
     if(err.name==='UnauthorizedError'){
        return res.send({
            staus: 401,
            message:'无效的token'
        })
    }
    res.send({
        status:500,
        message:'未知的错误'
    })
});

app.listen("3333",()=>{
    console.log("run as 3333")
})
```
### 使用pm2多进程部署express
```
npm install -g pm2
```
### 启动
```
pm2 start app.js 
pm2 start app.js -i max 根据有效CPU数目启动最大进程数目
pm2 start app.js -i 3 启动3个进程
pm2 start app.js -x 用fork模式启动 app.js 而不是使用 cluster
pm2 start app.js -x -- -a 23 用fork模式启动 app.js 并且传递参数 (-a 23)
pm2 start app.js --name my-app 启动一个进程并把它命名为 my-app
pm2 stop my-app   根据名字停止进程
```
