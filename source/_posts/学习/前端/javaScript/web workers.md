---
title: Web Workers
cover: /img/thumbnail/学习/前端/javaScript/js.png
thumbnail: /img/thumbnail/学习/前端/javaScript/js.png
date: 2022-03-15 20:13:50
toc: true
categories: 
- 学习
- 前端
tags: 
- javaScript
---

[Web Workers](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers)
Web Worker为Web内容在后台线程中运行脚本提供了一种简单的方法。线程可以执行任务而不干扰用户界面。此外，他们可以使用`XMLHttpRequest`执行 I/O  (尽管`responseXML`和`channel`属性总是为空)。一旦创建， 一个worker 可以将消息发送到创建它的JavaScript代码, 通过将消息发布到该代码指定的事件处理程序（反之亦然）。
<!--more-->
### <font color=#a862ea>专用worker</font>

一个专用worker仅仅能被生成它的脚本所使用。

#### <font color=#a862ea>生成一个专用worker</font>

```js
var myWorker = new Worker('worker.js');
```

#### <font color=#a862ea>专用worker中消息的接收和发送</font>

其中e.data是传过来的消息。

```js
myWorker.postMessage();//主线程发送给worker（main.js）

onmessage = function(e) { console.log(e.data) }//worker接收主线程的消息（worker.js）

postMessage();//worker发送给主线程（worker.js）

myWorker.onmessage = function(e) { console.log(e.data) }//主线程接收worker的消息（main.js）
```

#### <font color=#a862ea>终止worker</font>

```js
myWorker.terminate();//主线程终止

close();//worker终止 第一次的时候还是会接收到主线程传来的消息
```

#### <font color=#a862ea>处理错误</font>

当 worker 出现运行中错误时，它的 `onerror` 事件处理函数会被调用。它会收到一个扩展了 `ErrorEvent` 接口的名为 `error`的事件。

错误事件有以下三个用户关心的字段：

- `message`

  可读性良好的错误消息。

- `filename`

  发生错误的脚本文件名。

- `lineno`

  发生错误时所在脚本文件的行号。

#### <font color=#a862ea>生成subworker</font>

如果需要的话 worker 能够生成更多的 worker。这就是所谓的subworker，它们必须托管在同源的父页面内。而且，subworker 解析 URI 时会相对于父 worker 的地址而不是自身页面的地址。这使得 worker 更容易记录它们之间的依赖关系。

#### <font color=#a862ea>引入脚本与库</font>

Worker 线程能够访问一个全局函数`importScripts()`来引入脚本，该函数接受0个或者多个URI作为参数来引入资源；以下例子都是合法的：

```js
importScripts();                        /* 什么都不引入 */
importScripts('foo.js');                /* 只引入 "foo.js" */
importScripts('foo.js', 'bar.js');      /* 引入两个脚本 */
```

### <font color=#a862ea>共享worker</font>

一个共享worker可以被多个脚本使用——即使这些脚本正在被不同的window、iframe或者worker访问。

#### <font color=#a862ea>生成一个共享worker</font>

```js
var myWorker = new SharedWorker('worker.js');
```

一个非常大的区别在于，与一个共享worker通信必须通过端口对象——一个确切的打开的端口供脚本与worker通信（在专用worker中这一部分是隐式进行的）。

在传递消息之前，端口连接必须被显式的打开，打开方式是使用**onmessage**事件处理函数或者**start()**方法。

start()方法的调用只在一种情况下需要，那就是消息事件被**addEventListener()**方法使用。

在使用start()方法打开端口连接时，如果父级线程和worker线程需要双向通信，那么它们都需要调用start()方法。

```js
myWorker.port.start();  // 父级线程中的调用
port.start(); // worker线程中的调用, 假设port变量代表一个端口
```

#### <font color=#a862ea>共享worker中消息的接收和发送</font>

首先，当一个端口连接被创建时（例如：在父级线程中，设置onmessage事件处理函数，或者显式调用start()方法时），使用onconnect事件处理函数来执行代码。

```js
myWorker.port.postMessage();//主线程发送消息
myWorker.port.onmessage = function(e) {}//主线程接收消息
```

```js
onconnect = function(e) {//连接
  var port = e.ports[0];

  port.onmessage = function(e) {//子线程worker接收消息
    port.postMessage(workerResult);//子线程woker发送消息
  }
}
```

### <font color=#a862ea>嵌入式 worker</font>

目前没有一种「官方」的方法能够像 [``](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/script) 元素一样将 worker 的代码嵌入的网页中。但是如果一个 [``](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/script) 元素没有 `src 特性，并且它的` `type` 特性没有指定成一个可运行的 mime-type，那么它就会被认为是一个数据块元素，并且能够被 JavaScript 使用。「数据块」是 HTML5 中一个十分常见的特性，它可以携带几乎任何文本类型的数据。所以，你能够以如下方式嵌入一个 worker：

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8" />
<title>MDN Example - Embedded worker</title>
<script type="text/js-worker">
  // 该脚本不会被 JS 引擎解析，因为它的 mime-type 是 text/js-worker。
  var myVar = "Hello World!";
  // 剩下的 worker 代码写到这里。
</script>
<script type="text/javascript">
  // 该脚本会被 JS 引擎解析，因为它的 mime-type 是 text/javascript。
  function pageLog (sMsg) {
    // 使用 fragment：这样浏览器只会进行一次渲染/重排。
    var oFragm = document.createDocumentFragment();
    oFragm.appendChild(document.createTextNode(sMsg));
    oFragm.appendChild(document.createElement("br"));
    document.querySelector("#logDisplay").appendChild(oFragm);
  }
</script>
<script type="text/js-worker">
  // 该脚本不会被 JS 引擎解析，因为它的 mime-type 是 text/js-worker。
  onmessage = function (oEvent) {
    postMessage(myVar);
  };
  // 剩下的 worker 代码写到这里。
</script>
<script type="text/javascript">
  // 该脚本会被 JS 引擎解析，因为它的 mime-type 是 text/javascript。

  // 在过去...：
  // 我们使用 blob builder
  // ...但是现在我们使用 Blob...:
  var blob = new Blob(Array.prototype.map.call(document.querySelectorAll("script[type=\"text\/js-worker\"]"), function (oScript) { return oScript.textContent; }),{type: "text/javascript"});

  // 创建一个新的 document.worker 属性，包含所有 "text/js-worker" 脚本。
  document.worker = new Worker(window.URL.createObjectURL(blob));

  document.worker.onmessage = function (oEvent) {
    pageLog("Received: " + oEvent.data);
  };

  // 启动 worker.
  window.onload = function() { document.worker.postMessage(""); };
</script>
</head>
<body><div id="logDisplay"></div></body>
</html>
```

### <font color=#a862ea>其它类型的worker</font>

除了专用和共享的web worker，还有一些其它类型的worker：

- [ServiceWorkers](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorker_API) （服务worker）一般作为web应用程序、浏览器和网络（如果可用）之前的代理服务器。它们旨在（除开其他方面）创建有效的离线体验，拦截网络请求，以及根据网络是否可用采取合适的行动并更新驻留在服务器上的资源。他们还将允许访问推送通知和后台同步API。
- Chrome Workers 是一种仅适用于firefox的worker。如果您正在开发附加组件，希望在扩展程序中使用worker且有在你的worker中访问 [js-ctypes](https://developer.mozilla.org/en/js-ctypes) 的权限，你可以使用Chrome Workers。详情请参阅`ChromeWorker`。
- [Audio Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API#Audio_Workers) （音频worker）使得在web worker上下文中直接完成脚本化音频处理成为可能。

### <font color=#a862ea>worker中可用的函数和接口</font>

你可以在web worker中使用大多数的标准javascript特性，包括

- [`Navigator`](https://developer.mozilla.org/zh-CN/docs/Web/API/Navigator)
- [`XMLHttpRequest`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)
- [`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array), [`Date`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date), [`Math`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Math), and [`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)
- [`WindowTimers.setTimeout` (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) and [`WindowTimers.setInterval` (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/setInterval)

在一个worker中最主要的你不能做的事情就是直接影响父页面。包括操作父页面的节点以及使用页面中的对象。你只能间接地实现，通过[`DedicatedWorkerGlobalScope.postMessage` (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/DedicatedWorkerGlobalScope/postMessage)回传消息给主脚本，然后从主脚本那里执行操作或变化。