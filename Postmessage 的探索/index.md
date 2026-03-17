## Postmessage 的探索 
### 介绍

通常来说 postmessage 是实现不同窗体( `window` )的`跨源通信`，在浏览器里通过 `window.postMessage()` 方法来传递消息，这里的 window 是`目标窗体的引用`。

### 语法

#### 发送方

``` javascript
otherWindow.postMessage(message, targetOrigin, [transfer]);
```

#### 接收方

``` typescript
window.addEventListener("message", receiveMessage, false);

function receiveMessage(event: MessageEvent)
{
  var origin = event.origin;
  if (origin !== "http://example.org:8080")
    return;
  // ...
}
```

具体详情可见MDN： https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage

### Window 窗口

Window 可以是 window、window.opener、window.parent、window.top、iframe.contentWindow。

 `window.opener` ：如果当前窗口是由另一个窗口打开的，window.opener 保留了那个窗口的引用。如果当前窗口不是由其他窗口打开的，则该属性返回 null。

 `window.parent` ：返回当前窗口的父窗口对象。

* 如果一个窗口没有父窗口，则它的 parent 属性为自身的引用；
* 如果当前窗口是一个 `<iframe>`、`<object>` 或者 `<frame>`，则它的父窗口是嵌入它的那个窗口；

 `window.top` ：返回窗口层级最顶层窗口的引用。

 `iframe.contentWindow` ：通过元素属性 contentWindow 即可获取当前窗口。

``` javascript
var x = document.getElementsByTagName("iframe")[0].contentWindow;
//x = window.frames[0];
```

