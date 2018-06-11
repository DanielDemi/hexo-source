---
title: websock(AMQ)通信-前端
comments: true
toc: true
date: 2018-06-11 22:30:28
categories: websocket
tags: websocket
---
### 服务端和客户端之间的通信
前端开发经常会依赖后端，那么如果后端服务器还没做好推送服务器，那么前端该如何呢。
最简单的就是自己模拟一个服务器，用node来搭建，这边只简单介绍搭建的过程
#### node搭建服务器
##### 1.选取websocket插件进行开发
按照如下步骤进行开发
在命令台中输入以下命令
````
npm init -y 
npm i websocket -S
```
安装完websocket插件后就可以搭建服务器了，首先新建一个app.js，并且代码如下

```
var webSocketsServerPort = 3002
var webSocketServer = require('websocket').server
var http = require('http')

var server = http.createServer(function(req, res) {

})
server.listen(webSocketsServerPort, function() {
    console.log((new Date()) + "Server is listening on port " + webSocketsServerPort)
})

/**
 * websock server
 */
var wsServer = new webSocketServer({
    httpServer: server
})

wsServer.on('request', function(request) {
    var connection = request.accept(null, request.origin); 
    connection.on('message', function(message) {
        console.log('message');
    });

    connection.on('close', function(connection) {
        console.log('close');
    });

})
```
这样一个简易的服务器已经搭建完成。可以通过客户端进行测试(客户端代码见后面)

##### 2.选取faye-websocket插件进行开发
在命令台中输入以下命令

```
npm init -y 
npm i faye-websocket-S
```
这种方式更简单了。代码如下
```
var WebSocket = require('faye-websocket'),
    http      = require('http');

var server = http.createServer();

server.on('upgrade', function(request, socket, body) {
  if (WebSocket.isWebSocket(request)) {
    var ws = new WebSocket(request, socket, body);

    ws.on('message', function(event) {
      ws.send(event.data);
    });

    ws.on('close', function(event) {
      console.log('close', event.code, event.reason);
      ws = null;
    });
  }
});

server.listen(3002);
```

##### 3.选取socket.io插件进行开发
该插件依赖于express
命令如下
```
express -e node-js-socketio // express模板安装
npm i // 安装express插件
npm i socket.io -S // 安装socket.io
```
app.js文件改成如下代码
```
var app = require('express')()
  , server = require('http').createServer(app)
  , io = require('socket.io').listen(server);

server.listen(3002);

app.get('/', function (req, res) {
  res.sendfile(__dirname + '/client/index.html');
});

io.sockets.on('connection', function (socket) {
  socket.emit('news', { hello: 'world' });
  socket.on('my other event', function (data) {
    console.log(data);
  });
});
```
至此 服务器搭建已经完成(还有一种是基于stomp.js,目前还没弄清楚node如何搭建，等知道后在补充)


#### 客户端（即浏览器）调用服务器
##### 1.简单的demo如下
```
<html>

<body>
    <div id="output"></div>
    <script>
    function checkBrowser() {
        if (window.WebSocket) {
            log("This browser supports WebSocket!");
        } else {
            log("This browser does not support WebSocket.");
        }
    }

    function setup() {
        var wsServer = 'ws://localhost:3002';
        var ws = new WebSocket(wsServer);

        ws.onopen = function(e) {
            log("Connected to WebSocket server.", e);
            sendMessage("Conan");
        };

        ws.onclose = function(e) {
            log("Disconnected", e);
        };

        ws.onmessage = function(e) {
            log("RECEIVED: " + e.data, e);
            ws.close();
        }

        ws.onerror = function(e) {
            log('Error occured: ' + e.data, e);
        };

        var sendMessage = function(msg) {
            ws.send(msg);
            log("SEND : " + msg);
        }
    }

    function log(s, e) {
        var output = document.getElementById("output");
        var p = document.createElement("p");
        p.style.wordWrap = "break-word";
        p.style.padding = "10px";
        p.style.background = "#eee";
        p.textContent = "LOG : " + s;
        output.appendChild(p);
        console.log("LOG : " + s, e);
    }

    checkBrowser();
    setup();
    </script>
</body>

</html>
```
##### 2.基于express的socket.io的demo如下
```
<!DOCTYPE html>
<html>
<head>
<title>socket.io</title>
</head>
<body>
<h1>socket.io</h1>
<p>Welcome to socket.io</p>

<script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.1.1/socket.io.js"></script>
<script>
var socket = io.connect('http://localhost:3002');
socket.on('news', function (data) {
console.log(data);
socket.emit('my other event', { my: 'data' });
});
</script>
</body>
</html>
```

####3 基于AMQ的demo(目前还不知道服务器如何搭建，但客户端还是可以用的)
demo里放了3种模式，主要就是Queue和topic，消费和发布
Queue和topic的区别是一对多和多对多（QUEUE发送一个人消息之后，就消失了，而topic可以发送多个人）
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>

    <script type="stompjs"></script>
    <script type="text/javascript">
    // 3种模式   
    // QUeue是只有1对多模式，，topic多对多模式
    // 1.消息发布者模式
    var client = Stomp.overTCP('localhost', 61613);
    // uncomment to print out the STOMP frames

    var connectCallback = function(frame) {
        var subscription = client.subscribe('/topic/FirstQueue', onMessage);
        //subscription.unsubscribe();
    };

    var onMessage = function(message){
        if (message.body) {
          console.log("got message with body " + message.body)
        } else {
          console.log("got empty message");
        }
    };

    var errorCallback = function(error){
        console.log(error.headers.message);
    };

    client.connect('admin', 'admin', connectCallback,errorCallback);

    // 2.Queue消息消费者
    var Stomp = require('stompjs');

    // Use raw TCP sockets
    var client = Stomp.overTCP('localhost', 61613);
    // uncomment to print out the STOMP frames
    // client.debug = console.log;

    var connectCallback = function(frame) {
        var subscription = client.subscribe('/queue/FirstQueue', onMessage);
        //subscription.unsubscribe();
    };

    var onMessage = function(message){
        if (message.body) {
          console.log("got message with body " + message.body)
        } else {
          console.log("got empty message");
        }
    };

    var errorCallback = function(error){
        console.log(error.headers.message);
    };

    client.connect('admin', 'admin', connectCallback,errorCallback);
    // 3.topic 消息订阅者模式
    // Use raw TCP sockets
    var client = Stomp.overTCP('localhost', 61613);
    // uncomment to print out the STOMP frames
    // client.debug = console.log;

    var connectCallback = function(frame) {
        var subscription = client.subscribe('/topic/FirstQueue', onMessage);
        //subscription.unsubscribe();
    };

    var onMessage = function(message){
        if (message.body) {
          console.log("got message with body " + message.body)
        } else {
          console.log("got empty message");
        }
    };

    var errorCallback = function(error){
        console.log(error.headers.message);
    };

    client.connect('admin', 'admin', connectCallback,errorCallback);
    </script>
</body>
</html>
```