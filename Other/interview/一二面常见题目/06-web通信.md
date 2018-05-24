# Web通信

- 什么是同源策略，以及限制
- 前后端如何通信
- 如何创建Ajax
- 跨域通信的几个方式

## 什么是同源策略，以及限制

> 同源策略： 浏览器通信的一个安全机制，禁止你跨域的。

> 3个限制

- AJAX请求无法发送
- 无法操作另一个资源的DOM
- 无法获取cookie、localStorage

## 前后端如何通信

- Ajax : 仅仅支持同源通信
- WebSocket : 不受同源策略的限制
- CORS: 新的通信标准，也是可以跨域的。

##  如何创建一个AJAX

```
var xhr = new XMLHttpReqest() || .....

xhr.open('get','1.php')

xhr.send()

xhr.onreadystatechange = function(){
    if(xhr.status == 200 && xhr.readyState==4){
        xhr.responseText
    }
}


```

## 跨域通信的几个方式

- JSONP
- hash
- postMessage
- WebSocket
- CORS

> JSONP原理

JSONP的原理就是利用script的src可以跨域

前端写一段script去告诉后端callback名字，并且定义好这个函数执行什么逻辑

就等着资源完成后，后端调用这个函数那那一段代码了。

等于前端，定义了函数要做的逻辑、以及函数名

后端帮你调用了这个函数。


> CORS

CORS是用fetch来实现的

options预请求



浏览器会拦截ajax请求，如果它觉得这个请求是跨域的，会在请求头加一个origin

服务端有access-control-allow-origin设置哪些Origin是合法。