---
title: 使用 BroadcastChannel 进行跨窗口通信
date: 2024-09-25T02:59:29+08:00
updated: 2024-09-25T03:06:30+08:00
dg-publish: false
---

## 基本布局

```html
<!DOCTYPE html><html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <link rel="stylesheet" href="./index.css">
    <title>窗口通信</title>
  </head>
<body>
    <img draggable="false" class="card">
    <script src="./index.js"></script>
  </body>
</html>
```

首先定义一个 init 方法，用来初始化扑克牌，通过接收 url 中的 type 参数，显示不同的扑克牌

```js
const card = document.querySelector(".card")

function init(){
  const url = new URL(window.location);
  const type = url.searchParams.get("type") || 'Q'
  card.src = `./${type}.png`
}
init()
```

CSS 样式

```css
body{margin: 0;padding: 0;position: relative; width: 100%;height: 100%; overflow: hidden;}.card{width: 200px; height: auto;cursor: move;position: absolute;}
```

在浏览器访问 <http://localhost:8080/index.html?type=q>

## 拖动

然后定义扑克牌的拖动

```js
card.onmousedown = (e) => {
  let x = e.pageX - card.offsetLeft;
  let y = e.pageY - card.offsetTop;
  window.onmousemove = (e) => {
    const cx = e.pageX - x;
    const cy = e.pageY - y;
    card.style.left = cx + "px";
    card.style.top = cy + "px";
  };
  window.onmouseup = () => {
    window.onmousemove = null;
    window.onmouseup = null;
  }
}
```

## 统一坐标系

三个窗口分别是三张牌，需要让他们统一相对于屏幕的坐标系，写两个辅助函数

```js
/**
 * 获取浏览器窗口的工具栏和地址栏高度
 *
 * @returns 浏览器窗口的工具栏和地址栏的高度值
 */
 function barHeight(){
   return window.outerHeight - window.innerHeight;
 }
 
 /**
 * 将客户端坐标转换为屏幕坐标
 *
 * @param x 客户端x坐标
 * @param y 客户端y坐标
 * @returns 返回屏幕坐标数组 [screenX, screenY]
 */
 function clientToScreen(x,y){
  const screenX = x + window.screenX;
  const screenY = y + window.screenY + barHeight();
  return [screenX, screenY];
}

/**
 * 将屏幕坐标转换为客户端坐标
 *
 * @param x 屏幕横坐标
 * @param y 屏幕纵坐标
 * @returns 转换后的客户端坐标数组 [clientX, clientY]
 */
 function screenToClient(x,y){
  const clientX = x - window.screenX;
  const clientY = y - window.screenY - barHeight();
  return [clientX, clientY];
}
```

## 窗口通信

只有一个窗口在最开始需要显示扑克，可以在地址上加参数，隐藏的加上 <http://localhost:8080/index.html?type=q&hidden>

改写 init 函数

```js
function init(){
  const url = new URL(window.location)
  const type = url.searchParams.get("type") || 'Q'
  card.src = `./${type}.png`
  if (location.search.includes('hidden')) {
    card.style.left = '-1000px'
  }
}
```

现在需要在移动卡牌的时候让所有窗口的卡牌坐标一致（都相对屏幕在同一个位置）

通知其他窗口可以使用标签页间通信（同一电脑同一个浏览器、延迟低）和服务器 websocket（有延迟、可跨设备）

这里使用第一种

创建一个通道

```js
const channle = new BroadcastChannel("card")
```

通知其他窗口

```js
  window.onmousemove = (e) => {
    const cx = e.pageX - x;
    const cy = e.pageY - y;
    card.style.left = cx + "px";
    card.style.top = cy + "px";
    // 通知其他窗口
    channle.postMessage(clientToScreen(cx, cy))
  };
```

其他窗口接收消息

```js
channle.onmessage = (e) => {
  const clientPoints = screenToClient(...e.data);
  card.style.left = clientPoints[0] + "px"
  card.style.top = clientPoints[1] + "px"
}
```
