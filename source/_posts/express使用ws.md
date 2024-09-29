---
title: express使用ws
date: 2024-09-29 17:36:50
tags: 前端
categories: express
---
```
var express = require('express');
var expressWs = require('express-ws');
var app = express();
expressWs(app);

var PORT = 7777;
var clientObject = {};

app.ws('/', (client, req) => {
    var key = req.socket.remoteAddress + "_" + req.socket.remotePort;
    clientObject[key] = {
        cli: client,
        lastActiveTime: Date.now(), // 记录最后活跃时间
    };

    client.on('message', (message) => {
        client.send("收到你的消息了" + message);
        clientObject[key].lastActiveTime = Date.now(); // 更新最后活跃时间
    });

    client.on('close', (code, reason) => {
        console.log(`WebSocket close event code: ${code}, reason: ${reason}`);
        delete clientObject[key];
    });

    client.on('error', (err) => {
        console.error("WebSocket error observed:", err);
        delete clientObject[key];
    });
});

app.get('/', (req, res) => {
    res.send("hello 2023");
});

// 封装心跳逻辑
function setupHeartbeat() {
    setInterval(() => {
        var time = Date.now();
        for (var key in clientObject) {
            var client = clientObject[key].cli;
            var lastActiveTime = clientObject[key].lastActiveTime;
            
            // 检查最后一次活跃时间是否超过了1分钟（60000毫秒）
            if (time - lastActiveTime > 60000) {
                console.log('Client timed out due to inactivity:', key);
                // 关闭连接，删除
                client.close();
                delete clientObject[key];
                continue;
            }

            try {
                var sData = {
                    rspdata: {
                        heartBeat: time,
                        time: time,
                    },
                };
                client.send(JSON.stringify(sData));
            } catch (e) {
                console.error("Heartbeat error:", e);
                // 如果发送心跳失败，也关闭连接
                client.close();
                delete clientObject[key];
            }
        }
    }, 3000); // 每3秒发送一次心跳
}

app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
    setupHeartbeat(); // 设置心跳检测
});
```
