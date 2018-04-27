### 业务场景中如何选择适合的方式进行数据交换( form ,xhr, fetch, SSE, webstock, postmessage， web workers等)
```
前端经常使用的HTTP协议相关(1.0 / 1.1)method
GET ( 对应 restful api 查询资源, 用于前端从服务端取数据 )
POST(对应 restful api中的增加资源, 用于前端传数据到服务端)
PUT (对应 restful api中的更新资源)
DELETE （ 对应 restful api中的删除资源 )
HEAD ( 可以用于http请求的时间什么，或者判断是否存在判断文件大小等)
OPTIONS （在前端中常用于 cors跨域验证）
TRACE * (我这边没有用到过，欢迎补充)
CONNECT * (我这边没有用到过，欢迎补充)

Enctype ：指定将数据回发到服务器时浏览器使用的编码类型。用于表单里有图片上传。 
编码类型有以下三种： 
application/x-www-form-urlencoded： 在发送前编码所有字符（默认）。这是标准的编码格式。 
 var formData = new FormData(document.querySelector("#data2"));
$.ajax({
    method: "POST",
    contentType:'application/x-www-form-urlencoded;charset=UTF-8',
    url: "form_action.php",
    data: {username: "John", password: "Boston" }
}).done(function( msg )}{}
fetch("form_action.php", {
    method: "POST",
    credentials: 'include', //带上cookie
    headers: {
        "Content-Type": "application/x-www-form-urlencoded;charset=UTF-8"
    },
    body: "username=John&password=Boston"
})
.then(function(response){}
multipart/form-data： 不对字符编码，在使用包含文件上传控件的表单时，必须使用该值。
fetch("form_action.php", {
    method: "POST",
    headers: {
        "Content-Type": "multipart/form-data;charset=UTF-8"
    },
    body: formData
})

text/plain： 窗体数据以纯文本形式进行编码，其中不含任何控件或格式字符。 
$.ajax({
    method: "POST",
    contentType:'text/plain;charset=UTF-8',
    processData:false, //无需让jquery正处理一下数据
    url: "form_action.php",
    data: "我是一个纯正的文本功能!\r\n我第二行"
}).done(function( msg ) {}

application/json (ajax常用这种格式)
$.ajax({
  method: "POST",
  contentType:'application/json;charset=UTF-8',
  url: "form_action.php",
  data: JSON.stringify({username: "John", password: "Boston" })
}).done(function( msg ) {}
xml:
$.ajax({
    method: "POST",
    contentType:'text/xml;charset=UTF-8',
    // processData:false, //无需让jquery正处理一下数据
    url: "form_action.php",
    data: "<doc><h1>我是标签</h1><p>我是内容</p></doc>"
}).done(function( msg ) {
```



### 页面之间通信  <B>postMessage</B>
```
1. window.postMessage() 方法可以<B>安全地实现跨域通信</B>
2.主要用于两个页面之间的消息传送
3. 可以使用iframe与window.open打开的页面进行通信.
应用场景:
我们的页面引用了其他的人页面，但我们不知道他们的页面高度，这时可以通过window.postMessages 
从iframe 里面的页面来传到 当前页面.
<B>otherWindow.postMessage(message, targetOrigin, [transfer]);</B>
父域：
 window.post1.postMessage(date,'*');});
子域：
document.getElementById('sendbox2').addEventListener('click', function(){
  window.parent.post2.postMessage('收到来自左边ifarme的消息' + +new Date(),'*');
});



```


### 客户端与服务器双通信  <B>WebSocket</B>
```
https://github.com/websockets/ws/blob/master/doc/ws.md
https://github.com/websockets/ws
1. websocket 是个双向的通信。
2. 常用于应用于一些都需要双方交互的，实时性比较强的地方(如聊天,在线客服)
3. 数据传输量小
4. websocket 是个持久化的连接

服务器js：需要引入ws  npm install --save ws;
var WebSocketServer = require('ws').Server;
var wss = new WebSocketServer({port: 2000});
wss.on('connection', function(ws) {
    ws.send('服务端发来一条消息');
    ws.on('message', function(message) {
        //转发一下客户端发过来的消息
        console.log('收到客户端来的消息: %s', message);
        ws.send('服务端收到来自客户端的消息:' + message);
    });
    ws.on('close', function(event) {
        console.log('客户端请求关闭',event);
    });
});
前端：
<button id="btn">发点什么</button>
<div id="boxwarp"></div>
var ws = new WebSocket("ws://127.0.0.1:2000/");
document.getElementById('btn').addEventListener('click', function() {
  ws.send('cancel_order');
});
function addbox(msg){
  var box = document.createElement('div');
      box.innerHTML = msg;
  document.getElementById('boxwarp').append(box);
}
ws.onopen = function() {
    var msg = 'ws已经联接';
    addbox(msg);
    ws.send(msg);
};
ws.onmessage = function (evt) {
    console.log(evt.data);
    addbox(evt.data);
};
ws.onclose = function() {
   console.log('close');
   addbox('服务端关闭了ws');
};
ws.onerror = function(err) {
   addbox(err);
};
```
### 服务器到客户端的推送 – <B>Server-sent Events</B>这个是html5的一个新特性，主要用于<B>服务器推送消息到客户端</B>
可以用于监控、通知、更新库存之类的应用场景。
在携程运动项目中我们主要应用于线上被预订后通知下发通知到场馆的操作界面上的即时改变状态。
```
客户端：
var source = new EventSource('http://localhost/a/a.php');
source.onmessage = function(e){
  console.log(e.data)
};
php服务端：
header('Content-Type: text/event-stream');
header('Cache-Control: no-cache');
$time = date('Y-m-d H:i:s',time()+60*60*8);
$data = array(
    'id'=>1,
    'name'=>'中文',
    'time'=>$time
);
echo "data: ".json_encode($data)."\n\n";
flush();
```
